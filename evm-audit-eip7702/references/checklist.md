# EIP-7702 Security Checklist

Every item here is non-obvious — EIP-7702 changes the EVM trust model in ways that break assumptions baked into countless contracts.

## Authorization Tuple Hygiene

- [ ] **`chainId == 0` in auth tuple = signed for ALL chains**: EIP-7702 allows `chainId == 0` to indicate "any chain". A signature meant for one chain (or by a user who didn't realize) can be replayed on any other chain where the EOA has a matching nonce. Look for: any code or tooling that produces, validates, or relays SetCode authorizations — ensure `chainId` is a specific value, not 0, unless cross-chain delegation is explicit. [EIP-7702]

- [ ] **Auth nonce drift between sign and submit**: The auth tuple's `nonce` is the EOA's nonce at submission time. If the EOA does any other tx between signing and submitting, the auth is invalid (silently dropped). Worse, if the EOA does a tx that bumps nonce to the signed value LATER, the auth becomes replayable. Look for: relayers/sponsors that hold auth tuples beyond a few blocks. [EIP-7702]

- [ ] **Auth tuple mempool replay by other relayers**: Auth tuples are not bound to a specific tx — any builder/relayer who sees one in the mempool can include it in their own SetCodeTransaction. If the delegation does anything stateful (approves tokens, initializes admin), a competing relayer can hijack. Look for: delegations whose post-set-code call grants approval or admin to `tx.origin` / `msg.sender` of the SetCode tx. [Flashbots; EIP-7702]

- [ ] **`magic byte 0x05` signing context confusion**: 7702 auth tuples are signed with domain byte `0x05`. If a downstream signer reuses a personal-sign/EIP-191/EIP-712 flow without prepending the magic, signatures intended for other purposes may be coerced into auth tuples (or vice versa). Look for: wallet integrations that sign raw bytes for users — must explicitly distinguish 7702 auths. [EIP-7702]

- [ ] **Delegation persists across transactions**: The delegation lasts until the EOA signs another auth (to a different address OR `0x0` to clear). Many security models assume "one-shot delegation for one tx" — they don't get that. Look for: relayer UX that suggests delegation is per-tx; users will accumulate stale delegations. [EIP-7702]

## Storage Collisions in Delegated Code

- [ ] **OZ `Ownable` slot 0 collision**: A delegated implementation that inherits OpenZeppelin Ownable writes the owner to slot 0. If the EOA ever delegated to a different Ownable-based contract before, slot 0 holds garbage. New code reads it as "owner" and grants admin to a stale address. Look for: any Ownable / ReentrancyGuard / AccessControl mixin in a 7702 delegation target. Use namespaced storage (ERC-7201) instead. [OZ 5.x; ERC-7201]

- [ ] **ReentrancyGuard stuck state from prior delegation**: Prior delegate left `_status = _ENTERED` due to revert mid-function. New delegate's nonReentrant always reverts. Look for: lack of explicit slot reset on delegation or use of namespaced/transient storage. [OZ 5.x]

- [ ] **Proxy-style slot conventions assumed empty**: ERC-1967 implementation slot, beacon slot, admin slot — a 7702 delegation could write there too. If the same EOA later delegates to a proxy-aware implementation, it sees stale "implementation" pointers. Look for: any reuse of ERC-1967 slots in delegated code. [ERC-1967; ERC-7201]

- [ ] **No initializer guard, or initializer callable post-delegation**: Delegations have no constructor. If `initialize()` is `external` and not guarded by `_initialized`, anyone can re-init after the EOA delegates. Look for: missing `initializer` modifier or `disableInitializers` calls. Note: `_initialized` itself lives in storage and may be polluted from a previous delegation. [OZ 5.x Initializable]

- [ ] **`immutable` variables wrong for the EOA**: `immutable` and `constant` are bytecode-embedded values from the *implementation contract's* deployment. A delegated EOA inherits those — `address(this)` is the EOA, but `IMMUTABLE_OWNER` is the deployer of the implementation. Look for: implementations that bake the deployer into immutables and expect "owner == address(this).owner". [Pashov]

- [ ] **ERC-7201 namespaced storage REQUIRED for libraries used in 7702 targets**: Libraries (OZ, Solady) that aren't ERC-7201-namespaced are unsafe to compose in a 7702 implementation. Look for: mixin inheritance in delegation targets — every base contract should use namespaced storage. [ERC-7201]

## Re-Delegation & Sweeper Attacks

- [ ] **Stored approvals after re-delegation**: Old delegation granted `IERC20.approve(router, max)` and held funds in the EOA. EOA owner re-delegates to a sweeper contract; sweeper now controls those approvals AND the EOA's balance. Look for: 7702 wallets that grant token approvals — clear them on de-delegation. [Solodit; EIP-7702]

- [ ] **Sweeper as default delegation post-phishing**: A phishing site convinces the user to sign a delegation to a sweeper contract; sweeper drains every token and ETH. Once signed, the auth tuple is broadcastable forever within nonce window. Look for: UX that lets users sign auth tuples without showing the target address + simulating the result. [EIP-7702 wallet UX]

- [ ] **`SELFDESTRUCT` from delegated code drains the EOA**: A malicious delegation can `selfdestruct(attacker)` — under Cancun rules selfdestruct only sends ETH (no code removal), but that's still a full drain of the EOA's native balance. Look for: any third-party delegation target that includes selfdestruct. [EIP-6780; EIP-7702]

- [ ] **Recovery flow assumes delegation can be cleared by anyone**: To clear delegation, the EOA must sign a tuple to `address(0)`. If the private key is lost, the EOA is permanently delegated. Recovery contracts that rely on a "guardian set" to reset delegation are wrong — guardians can call into delegated code, but cannot SIGN a new auth tuple. Look for: social-recovery 7702 wallets claiming key-loss recovery. [ERC-4337 vs 7702]

## Broken `tx.origin == msg.sender` Idioms

- [ ] **"EOA-only" guard via `tx.origin == msg.sender`**: This idiom (used to block contract-based flash-loan or composition attacks) no longer means "human user". A 7702-delegated EOA executes code AND has `tx.origin == msg.sender`. Look for: this exact check; either remove it (the protection was always limited) or upgrade to `EXTCODESIZE == 0` (still imperfect — pre-7702 EOAs only). [EIP-7702]

- [ ] **Airdrop / mint "no contracts" gate fails**: Many airdrops gate with `require(msg.sender == tx.origin)`. Post-7702, a sybil farmer delegates an EOA to a multi-claim contract and farms all addresses atomically. Look for: airdrop / mint contracts using this gate. [Solodit]

- [ ] **`EXTCODESIZE(account) == 0` no longer implies "EOA without code"**: A 7702-delegated EOA has nonzero `EXTCODESIZE` (returns the size of the delegation prefix `0xef0100||addr` = 23 bytes). But conversely, an EOA *between* delegations has size 0. Neither check distinguishes "human-controlled" from "contract-like". Look for: any reliance on EXTCODESIZE for trust decisions. [EIP-7702]

- [ ] **Signature verification via `ecrecover` on a 7702 EOA**: A delegated EOA may implement ERC-1271 `isValidSignature`. Code that ONLY uses `ecrecover` won't see those signatures, even if the wallet says "I signed this". Look for: any signature-gated function (permit, governance vote, off-chain order) that doesn't fall back to ERC-1271 on contract-shaped accounts — now must include 7702 EOAs too. [ERC-1271; EIP-7702]

- [ ] **Native ETH transfers to EOAs now execute code**: `addr.call{value:1 ether}("")` to an EOA was previously safe from reentrancy (no code, no callback). With 7702, the EOA may be delegated to code that runs on receive. Look for: every `.call{value:}`, `.transfer`, `.send` to user-supplied addresses — needs reentrancy guards now. [EIP-7702; cross-ref evm-audit-reentrancy]

## Sponsored-Transaction & Relayer Patterns

- [ ] **Sponsor pays gas, attacker chooses target call**: A naive sponsor signs auth tuples for users; user-signed call data is then executed by the delegation. If the call data isn't constrained, attacker uses sponsor's auth to invoke arbitrary functions on the delegation (e.g., `setApprovalForAll(attacker, true)`). Look for: sponsor flows that don't bind the auth tuple to a specific tx hash or expected calldata. [Pashov]

- [ ] **Relayer front-run reorders auth tuples**: A SetCodeTransaction can include MULTIPLE auth tuples (for batch onboarding). A malicious relayer reorders them or omits some. Look for: protocols that broadcast multi-auth flows where order matters (e.g., parent auth must land before child). [EIP-7702]

- [ ] **Gas-griefing via delegation that consumes 2300 gas**: Old `.transfer()`/`.send()` (2300 gas) to a 7702-delegated EOA reverts because the delegation prologue costs more. This breaks payouts to anyone who's delegated. Look for: payouts via `.transfer()` (already deprecated, but doubly broken now). [EIP-7702]

- [ ] **`msg.value` accounting when delegation triggers reentry**: A delegation that executes on inbound ETH may reenter the payer mid-batch. Look for: batched ETH payouts (airdrops, multi-claim) without per-recipient `nonReentrant`. [EIP-7702]

- [ ] **Sponsor account is itself 7702-delegated**: If your sponsor EOA is delegated, the EOA's code runs first on incoming refunds — potentially unrelated to your sponsor logic, draining sponsor balance. Look for: relayer infra that uses 7702 wallets as the sponsor EOA. [Pashov]

## Auditor-Side Tooling Gaps

- [ ] **Slither / Mythril don't model 7702 yet**: Static analyzers treat EOAs as code-free. Any "no reentrancy because target is EOA" inference is wrong. Look for: tooling output you trust that predates 7702 modeling. [ToB]

- [ ] **Foundry forks don't replay SetCode txs faithfully across forks**: Replaying mainnet txs in a fork may strip 7702 auths if the fork's chainId differs and the auth wasn't `chainId == 0`. Tests that pass in fork may fail (or worse, succeed differently) on real mainnet. Look for: integration tests using `vm.createFork` against 7702-affected protocols. [Foundry]

- [ ] **Block explorers may not show delegation prefix**: An EOA with active delegation shows zero/odd bytecode on some explorers. Confirm using `eth_getCode` directly; bytecode begins with `0xef0100` followed by the 20-byte delegate address. [EIP-7702]
