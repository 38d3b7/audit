# Reentrancy Security Checklist (Beyond `nonReentrant`)

Every item here is non-obvious — single-function classic reentrancy with a `nonReentrant` modifier is excluded.

## Read-Only Reentrancy

- [ ] **View function consumed by integrators mid-callback**: A view function (`getPrice`, `getReserves`, `convertToAssets`, `pendingRewards`, `totalAssets`) returns state that is temporarily inconsistent during an external call. Integrators who read it during a callback get a value that reflects a half-completed update. Look for: any external call (token transfer, `.call`, callback) that happens AFTER a partial state mutation, where the contract also exposes a view of that state. The view function does NOT need `nonReentrant`, but it must read the **post-effects** state, not mid-effects. [Curve Finance July 2023 incident; Sherlock; ToB]

- [ ] **Curve-style pool with `remove_liquidity` reentry**: ETH refunds in `remove_liquidity` happen before LP totalSupply is decremented. A reentering contract sees inflated `get_virtual_price()` and can drain integrating lenders. Even if your contract isn't Curve, mimicking the pattern (return ETH/native before decrementing supply) is the bug. Look for: native ETH refunds with `address(this).balance`-based pricing, before state finalization. [Solodit; Cyfrin Updraft]

- [ ] **Oracle adapters trusting `latestAnswer` mid-tx**: An oracle adapter that wraps an on-chain pool's spot price returns stale or manipulated values if read during a callback inside that pool. Look for: TWAP/spot adapters that don't enforce a same-block guard on the pool they read. [SigmaPrime oracles]

- [ ] **Vault views (`totalAssets`, `convertToShares`) during deposit callback**: ERC4626 vault that pulls tokens via `safeTransferFrom` (which can trigger ERC777 hooks) BEFORE recording the deposit. Mid-hook, integrators see `totalAssets` already increased but `totalSupply` not yet — share price spikes briefly. Look for: deposit/withdraw flows that touch external tokens before share accounting. [ERC4626 primer; Solodit]

- [ ] **Lending health views (`getAccountLiquidity`) during liquidation refund**: Liquidator receives bonus collateral in a callback-capable token before debt is fully zeroed. Mid-callback, the account looks more solvent than it is. Look for: liquidation flows that transfer collateral before clearing debt counters. [Dacian lending]

- [ ] **`balanceOf(self)` invariants broken by mid-callback withdrawals**: A contract that uses `address(this).balance` or `token.balanceOf(address(this))` inside a view, where another function can pull tokens out via a callback. Look for: `internalBalance = balance - reserved` style views. [beirao V-01/V-02]

## Cross-Function & Cross-Contract Reentrancy

- [ ] **`nonReentrant` on the same function only**: Modifier applied to `withdraw()` but not to `claimRewards()` which reads/writes the same `userInfo[msg.sender]`. Attacker reenters via the unguarded function. Look for: state shared between guarded and unguarded functions; multi-function guard requires the SAME lock variable across all entry points. [beirao G-19]

- [ ] **Shared lock between proxy and implementation in upgradeable contracts**: After upgrade, the new implementation declares a fresh `_reentrancyStatus` at the same slot — but old code already wrote `_ENTERED`. Lock is stuck or bypassed. Look for: upgrades that don't preserve the OpenZeppelin `ReentrancyGuardUpgradeable` storage slot. [OZ 5.x]

- [ ] **Cross-contract reentry via shared accounting (router/pool pair)**: A router transfers tokens then calls the pool's `swap()`; the pool calls back into the router. Each contract has its own `nonReentrant`, but the SHARED state (router's `pendingFees`) is mutated twice. Look for: pairs of contracts where one calls the other and both mutate the same off-chain-tracked counter. [Solodit; Pashov]

- [ ] **Re-entry across different proxies sharing the same implementation**: Two proxies (e.g. two vaults) point at the same implementation. `_reentrancyStatus` lives in each proxy's storage, so a callback into the OTHER proxy is not blocked. If they share an underlying pool, you can drain. Look for: factories that deploy proxies of the same implementation handling the same underlying. [ToB]

- [ ] **Reentrancy via `delegatecall`-based plugin**: Module/plugin invoked via `delegatecall` runs in the parent's storage. An attacker-installed plugin can re-enter the parent and bypass `nonReentrant` because the lock is set BY the same address pre-delegatecall. Look for: any modular/plugin pattern using `delegatecall` to user-installed code. [ERC-7579; Pashov]

- [ ] **Sister-function not guarded after refactor**: Codebase grew, new entry point added (`zapIn`, `migrate`, `flashEnter`) that bypasses the originally-guarded `deposit`. Look for: any function that ends up at the same internal `_doStuff()` but lacks the modifier. [C4]

- [ ] **Lock guards `external` but not `public`**: `_doStuff` is `public`, called both by the guarded external and by another contract directly. Look for: `public` internal helpers reachable from outside. [Slither/ToB]

## EIP-1153 Transient Storage Lock Pitfalls

- [ ] **Forgetting to clear (`tstore(slot, 0)`) at end of function**: Transient storage clears at end of TRANSACTION, not function. If a function `tstore(LOCK, 1)` but reverts before `tstore(LOCK, 0)`, the next call within the SAME transaction (e.g. via a multicall) sees the lock still set, breaking flows. Conversely, code that *expects* it to clear between calls in the same tx will silently fail. Look for: `tstore`/`tload` lock patterns; ensure `tstore(slot, 0)` runs in a `finally`-equivalent (after all logic, before return). [EIP-1153; OZ 5.x ReentrancyGuardTransient]

- [ ] **Same transient slot used by multiple unrelated functions**: Slot collision in transient storage is just as dangerous as in regular storage. A library that uses transient slot `0x0` collides with the contract's lock at slot `0x0`. Look for: hardcoded transient slot constants (`bytes32 constant SLOT = 0x...`) duplicated across files. [EIP-1153]

- [ ] **Transient lock bypass via top-level multicall**: A multicall wrapper enters function A (sets transient lock), exits (clears lock), then enters function B in the same tx. If your protection relies on "if this tx already entered X, block Y", transient storage will NOT preserve that across the call boundary. Look for: protections like "only one deposit per tx" implemented via tload — they fail if both deposits clear the lock between them. [EIP-1153]

- [ ] **Mixing storage and transient locks in upgrade**: Pre-upgrade code uses `_reentrancyStatus` in storage; new code uses transient. Old storage slot is now garbage but reads still return old values, locking the contract forever after one mid-upgrade entry. Look for: migrations from `ReentrancyGuard` → `ReentrancyGuardTransient` without clearing the old slot. [OZ 5.x]

- [ ] **`tload` returns 0 on chains without EIP-1153**: Deploying to a chain that doesn't support Cancun opcodes silently bypasses the lock (returns 0, treated as "not entered"). Look for: contracts using transient locks deployed to non-Cancun L2s. [chain-specific; EIP-1153]

- [ ] **Transient storage assumed persistent across `CALL`**: Transient storage IS visible across CALL (and CREATE) — but NOT across DELEGATECALL boundaries differently than expected. Read the spec: transient storage is keyed by the **address whose code is executing**, but in delegatecall the address is the caller's. A subtle assumption flip can leak lock state to nested contexts. Look for: `tstore` inside a function that may be reached via delegatecall from a different address. [EIP-1153]

## Callback-Bearing Token Reentry

- [ ] **`safeMint` / `_safeMint` triggers `onERC721Received` before `_balances` increment**: OpenZeppelin's `_safeMint` calls the receiver hook AFTER the balance update, but custom forks frequently inline the hook first. A malicious receiver can re-enter the minter while balance is stale. Look for: any `_safeMint` reimplementation, especially in soulbound / staking-NFT contracts. [Solodit]

- [ ] **`ERC1155.safeTransferFrom` reenters the operator hook**: Same shape as ERC721 but with `onERC1155Received` and `onERC1155BatchReceived`. The batch variant lets a malicious receiver reenter once per token id in the batch. Look for: batch transfers in markets, vesting, airdrops. [C4]

- [ ] **ERC777 `tokensReceived` hook on `transfer` (not just `send`)**: ERC777 is ERC20-compatible but adds a callback to the recipient on `transfer`. Any contract that calls `IERC20.transfer(user, amt)` where the token is ERC777 hands control to `user`. imBTC and other ERC777-style tokens have caused historical drains. Look for: tokenlists allowing arbitrary ERC20s, or pools/vaults accepting whitelisted tokens that include ERC777. [weird-erc20; Uniswap V1 imBTC incident]

- [ ] **Approval hooks (`ERC777 tokensToSend`)**: ERC777 also calls the sender hook on outbound transfer, BEFORE the balance is decremented. A malicious sender contract can reenter and double-spend. Look for: contracts that pull tokens from users via `transferFrom` without `nonReentrant`. [weird-erc20]

- [ ] **Hook in `safeApprove`/permit2 callbacks**: Permit2 batch transfer with witness can invoke arbitrary callback selectors. Look for: integration with Permit2's `SignatureTransferDetails` where `to` is attacker-controlled. [Permit2]

- [ ] **NFT marketplace bid acceptance reentry**: Marketplace transfers NFT to buyer via `safeTransferFrom`. Buyer's `onERC721Received` reenters and cancels the listing, leaving the marketplace holding both the NFT AND having paid the seller. Look for: listing-state cleanup AFTER `safeTransferFrom` to buyer. [Solodit; C4]

## Subtle / Modern Patterns

- [ ] **Reentrancy via `staticcall` to a contract that uses `RETURNDATASIZE` tricks**: A view called via staticcall can't write state, but can return huge buffers to grief the caller. Combined with a wrapping contract that copies returndata into memory for inspection, this becomes a gas-griefing DoS that looks like reentrancy. Look for: cross-contract view aggregators (`Multicall3`, `Lens`). [beirao E-04]

- [ ] **Reentrancy through fallback-handler on modular accounts**: ERC-7579/6900 accounts route unknown selectors to an installed fallback handler. The handler can be attacker-controlled and re-enter the account. Look for: modular accounts with a default fallback module and no per-handler lock. [ERC-7579]

- [ ] **Reentry via signature-based meta-tx in the same block**: Meta-tx relayer executes `permit` then `transferFrom` for user A. If `permit` reverts mid-execution after a callback, A's nonce may have already incremented in transient state. Look for: combined permit + action atomic flows without per-step state checkpointing. [EIP-2612; Sherlock]

- [ ] **Reentrancy via EIP-7702 delegated EOA**: An EOA delegated to malicious code can return-call into your contract after you `transfer` ETH to it. Native ETH transfers via `.call{value:}` to an EOA *previously assumed safe* now hand it execution. Look for: any pattern that pays ETH to user-supplied addresses without `nonReentrant`, post-Pectra. [EIP-7702]

- [ ] **Cross-chain re-entry via message-passing bridges**: A LayerZero/CCIP `lzReceive` handler reenters when the destination chain has not yet finalized inbound nonce. Bridges with optimistic execution can deliver duplicate messages during reorg. Look for: bridge receivers without a per-nonce executed flag set BEFORE the action. [LayerZeroV2 checklist]

- [ ] **`call` to user address with non-zero value triggers receive() execution**: Even `addr.call{value: x}("")` can execute attacker code if `addr` is a contract (or a 7702-delegated EOA). The "I'm just refunding ETH" mental model is wrong. Look for: any value-bearing `.call` to user-supplied addresses without re-entrancy protection. [beirao E-07]
