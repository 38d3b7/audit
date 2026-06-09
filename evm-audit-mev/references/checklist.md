# MEV Security Checklist

Every item here is non-obvious — generic "use a private mempool" advice is excluded; this is about contract-side defenses.

## Missing Slippage / Deadline Protection

- [ ] **Function takes user calldata but no `minOut` / `maxIn`**: Any swap/burn/redeem path that converts asset A → B without a `minAmountOut` parameter is sandwichable. Look for: every `swap`, `redeem`, `unwrap`, `withdrawAll`, `migrate` — must accept a user-supplied minimum. [Dacian slippage]

- [ ] **`minOut == 0` allowed and used by frontends**: Even if the parameter exists, if zero is accepted, integrators (including the project's own UI) pass zero and users are sandwiched. Look for: `require(minOut > 0)` or explicit oracle-derived floor on the contract side. [Dacian slippage]

- [ ] **No `deadline` parameter, or `deadline = type(uint256).max`**: Without a deadline, a validator can hold a tx in the mempool until price moves favorably for them, then include it. Look for: missing `require(block.timestamp <= deadline)`. [Uniswap V2; Dacian]

- [ ] **Slippage check against post-trade state, not pre-trade quote**: `require(amountOut >= minOut)` where `amountOut` comes from the same swap call IS correct; but `require(reservesAfter > reservesBefore * threshold)` is not — sandwicher can satisfy that with their backrun. Look for: invariants on pool state instead of on user-received amount. [Solodit]

- [ ] **Multi-hop slippage applied only end-to-end**: A 3-hop swap with end-to-end minOut still allows intermediate pools to be sandwiched, leaking value at each hop. Look for: integrators that should be using single-pool routes with per-hop slippage. [Dacian]

- [ ] **`expectedShares` for ERC4626 deposit missing**: ERC4626 standard allows `deposit(assets, receiver)` without a shares-min check; a frontrunner who inflates assets-per-share between user's `previewDeposit` and execution makes the user receive far fewer shares. Look for: deposit/mint paths where caller doesn't pass / contract doesn't enforce a min/max shares. [ERC4626 primer]

## Frontrun-Mint / Allowlist / Rarity

- [ ] **NFT mint with pseudo-random rarity in same tx**: `tokenURI` or rarity derived from `block.prevrandao` / `block.timestamp` / `blockhash` in mint tx. Validator/builder picks the block to mint, attacker mints only on profitable blocks. Look for: any randomness sourced from block fields used for value-determining mints. [Solodit; SWC-120]

- [ ] **Commit-reveal without enforced commit-to-reveal gap**: Commit happens, reveal can happen in same block or next. An MEV searcher mempool-sniping the reveal can react. Look for: `reveal()` callable in same block as commit; require minimum block delta. [VRF best practices]

- [ ] **Allowlist mint with signature, signature reusable across rounds**: Signature for round N includes `(user, round, nonce)`; round-end is determined by `block.number`. Frontrunner sees user's tx and snatches the allowlist slot with the same signature replayed (if anyone-can-mint-on-behalf is allowed). Look for: signatures that don't bind `tx.origin` or `msg.sender`. [C4]

- [ ] **First-N-buyers bonus**: Anything with "first 100 buyers get 2x rewards" is a MEV race — only bots win, users always lose. Look for: rank-based incentives, replace with random selection from sign-up pool. [Pashov]

## Backrunnable / Sandwichable State

- [ ] **Oracle update + dependent action in separate txs (mempool window)**: Liquidatable account is "borderline" until the oracle update lands. Mempool watchers race the oracle update tx to liquidate at the new price. Defense: combine via oracle-callback, EIP-712 signed quotes, or push-based liquidator-only flows. Look for: protocol that lets liquidations happen on permissionless writes after price-feed tx. [Dacian liquidation; SigmaPrime]

- [ ] **`rebase` / `accrueInterest` callable by anyone for free**: A bot calls `rebase()` to trigger reward distribution in the same block it backruns to capture the rebase delta. Look for: stateful state-updating functions without keeper-only modifier or token-fee for the caller. [Dacian]

- [ ] **Public function whose only purpose is to update state another function reads**: If `updatePriceCache()` is public, MEV bots will call it inside their own bundle. Often harmless, but if the cache write is irreversible (e.g., starts a cooldown) bots can DoS legitimate flows. Look for: stateful "view-like" helpers. [Sherlock]

- [ ] **Auction settle-bid race**: First valid bid wins, ties broken by tx position. Bidders can frontrun each other's "I bid X" txs by replicating with `X + 1`. Look for: open-bid auctions without sealed-bid / commit-reveal. [C4]

- [ ] **Vault deposit-then-action**: User deposits to a vault then immediately swaps from the vault's new state. A frontrunner deposits first, sandwiches the swap. Look for: protocols that mix deposit and action in batched/multicall flows visible in mempool. [Pashov]

## JIT (Just-In-Time) Liquidity

- [ ] **CLM / V3 pool allows tight-range LP for one block**: JIT bot watches a large swap entering mempool, adds concentrated liquidity at exact swap price, captures fees, removes after swap. Reduces fees for honest LPs. Look for: pools without minimum LP lock duration or per-block LP cap. [Hacken UniV4]

- [ ] **V4 hooks that ratchet fees DOWN for new LPs are JIT-friendly**: Custom hook reduces fee for "new" LPs to attract liquidity; JIT bots qualify. Look for: hook-based fee schedules that don't include time-of-position checks. [Hacken UniV4]

- [ ] **Vault accepts deposit just before reward distribution**: A bot deposits 1 wei before `claim()` and dilutes existing holders. Look for: rewards distributed per share without minimum-holding period or snapshot-at-block-N. [ERC4626; Dacian staking]

## Atomic Backrun / Griefing

- [ ] **Function reverts on no-profit, leaving caller paying gas**: A "permissionless" liquidator function that reverts unless profit > threshold = griefable. Bots will call to drain user's gas with no-ops. Look for: keeper paths that revert on small differences. [Solodit]

- [ ] **`returndata` consumed without size limit (return-data bomb)**: Caller `.call`s a callback whose contract returns massive bytes, forcing the caller to copy them. Combines with backrunnable patterns to make legitimate users OOG. Look for: any `.call()` where return data is consumed without `_assertReturnDataSize` or assembly truncation. [beirao E-04]

- [ ] **Atomic-arb griefing via flashloan + small swap**: Attacker takes a flashloan, executes a backrun, repays; gas cost paid from the protocol's own surplus accumulation. Look for: protocols that subsidize gas of "arbitrage helper" callers. [Flashbots]

## Bundler / Builder / Private Mempool Trust

- [ ] **Code assumes private mempool ⇒ no sandwich**: "We use Flashbots Protect" is not a contract-side defense. The same mempool tx may be revealed if the user's wallet falls back to public mempool. Look for: any documented reliance on private RPC as security control. [Flashbots]

- [ ] **Bundler can reorder UserOps in 4337 bundle**: 4337 entrypoint executes bundle in order chosen by bundler. UserOps that depend on each other's state must declare proper nonce sequencing. Look for: paymaster economics that assume specific order. [ERC-4337 v0.7]

- [ ] **RFQ-style quote not bound to block**: User signs RFQ quote; market maker delays execution to a more favorable block. Look for: RFQ signatures without `validUntil` block (not just timestamp). [Solodit]

- [ ] **`block.coinbase` payment for MEV gives builder unbounded reward**: A contract that pays `block.coinbase` for inclusion subsidizes builders disproportionately, creating incentive for them to grief legitimate users. Look for: `block.coinbase.transfer` patterns. [Flashbots]

## Cross-Domain MEV

- [ ] **L2 sequencer can frontrun centrally**: On Arbitrum, Optimism, Base, the sequencer orders txs and has private visibility into the mempool. "Mempool-private" L2 strategies don't apply. Look for: protocols assuming public mempool MEV doesn't exist on L2. [Arbitrum docs; chain-specific]

- [ ] **Cross-chain intent solvers act as builders**: Solvers in ERC-7683 systems see all pending orders before fills land; they can extract MEV both on origin and destination. Look for: solver flows with both buy and sell sides visible. [ERC-7683; cross-ref evm-audit-intents]

- [ ] **Time-bandit reorg attacks on prob-finality chains**: An MEV-rich block on BSC/Polygon can be reorged by a malicious validator who builds a competing chain. Look for: protocols on prob-finality chains with single-block confirmation windows for value-extractive actions. [chain-specific]
