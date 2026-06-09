# Cross-Chain Intents Security Checklist (ERC-7683 et al.)

Every item here is non-obvious — generic bridge issues are in `evm-audit-bridges`; this is intent / solver specifics.

## Order Construction & Signing

- [ ] **Order hash omits `originChainId` or `destinationChainId`**: ERC-7683 `ResolvedCrossChainOrder` includes both chain IDs but some implementations EIP-712-hash only the user-facing fields. A solver can replay an order intended for chain A → B as A → C if the user's signed digest didn't bind C. Look for: EIP-712 typehash for the order — must include `originChainId` AND `destinationChainId` AND `fillDeadline` AND `nonce`. [ERC-7683]

- [ ] **Order nonce per-user, not per-(user, origin)**: User signs an order on chain A; same nonce reused on chain B if the order schema doesn't include chain. Look for: nonce stored as `mapping(address => uint256)` without chain dimension. [ERC-7683]

- [ ] **`openDeadline` vs `fillDeadline` confusion**: `openDeadline` is until when the user's order can be opened on origin; `fillDeadline` is until when a solver can claim it. Misnaming or swapping these lets a solver claim on a stale order. Look for: only ONE deadline check in `open()` / `fill()`. [ERC-7683]

- [ ] **`minReceived` / `maxSpent` validation missing or off-by-token**: User signs "I want at least X of token Y" but solver delivers token Z of equal USD value, claiming order satisfied. Look for: `Output` struct comparison that doesn't strictly match `token` AND `amount` AND `recipient`. [ERC-7683]

- [ ] **Signature schema doesn't bind solver**: Order signature MUST NOT bind to a specific solver (else exclusivity replaces an open auction), but MUST allow optional exclusivity period with a single solver. Implementations that always commit to one solver disable competition. Look for: `filler` field in the signed struct that isn't optional, or exclusivity logic that doesn't honor a `customAuthority`. [Across]

## Origin Settler — Releasing User Funds

- [ ] **Fill proof verified by trusted oracle, not by destination settler's commitment**: Origin releases user funds to solver after verifying a signed "fill happened" attestation. If the attestation only proves the solver SAID they filled (not that the destination contract recorded it), a malicious solver attesting their own fill drains origin. Look for: attestation schemes that don't bind to a tx hash AND a destination-side state commitment. [ERC-7683; LayerZero V2]

- [ ] **Replay across bridges**: Order can be settled via either bridge A or bridge B; once settled via A, attacker replays via B (different message format). Look for: `nonce`/`orderId` not consumed on first settle. [Across]

- [ ] **`orderId` collision via predictable preimage**: `orderId = keccak256(abi.encode(user, nonce))` — two users with the same nonce collide. Look for: orderId derivations that don't include order data. Use full struct hash. [ERC-7683]

- [ ] **Partial-fill accounting allows overdraw**: An order allows partial fills (e.g., 50% now, 50% later). The remaining-amount counter underflows or rounds in the solver's favor. Look for: arithmetic in fill paths without `require(remaining >= amount)` before decrement. [Pashov]

- [ ] **Refund path callable while pending fill**: User-initiated refund after `fillDeadline` doesn't check whether a fill is in-flight (cross-chain message in transit). Solver lands the fill AFTER user refunds = order paid twice. Look for: refund functions that only check timestamp, not a pending-fill flag. [Solodit]

## Destination Settler — Disbursing to User

- [ ] **Fill records solver claim AFTER paying user, no atomicity**: `transferFrom(solver, user, amount)` succeeds; then `_recordFill(orderId, solver)` reverts (e.g., OOG, bad input). User got paid but origin still owes solver — solver later claims again. Look for: external call before state write in fill flow. [ERC-7683]

- [ ] **`fill()` doesn't bind to origin chain ID**: A solver fills "the same orderId" from a different origin chain claim and double-collects. Look for: `fill(orderId, ...)` that doesn't include `originChainId` in fill-record key. [ERC-7683]

- [ ] **Fill price oracle stale or manipulable**: If the order specifies "user wants ETH-equivalent of $X" the destination settler reads ETH price from a pool — same MEV/oracle issues as DeFi (see `evm-audit-oracles`). Look for: price reads inside fill paths. [Across]

- [ ] **Permit2 batch fill with witness, wrong witness type**: ERC-7683 reference implementations use Permit2 with custom witnesses to bind transfers to order details. If witness type hash is wrong, signature verifies against an attacker-controlled witness. Look for: hardcoded witness type strings — must match exactly. [Permit2; Uniswap]

## Solver Economics & Griefing

- [ ] **Solver pre-funds destination, can be DoS'd by stale orders**: A malicious "user" signs orders that can never be filled profitably; solver wastes gas attempting. Look for: lack of pre-trade simulation gating, or absence of solver allowlist for high-value lanes. [Solver UX]

- [ ] **Order eligible to multiple solvers in same block**: All solvers race to fill, one wins, rest pay gas for nothing. Without exclusivity windows, solver economics break. Look for: missing exclusivity period or per-solver Dutch auction logic. [UniswapX; Across]

- [ ] **Solver's settlement claim can be front-run**: Solver fills destination, then submits settlement claim on origin. If the claim is permissionless and identity-bound to the solver address (not their signature), another party can replay it. Look for: `settle(orderId, solver, proof)` callable by anyone, paying `solver` — but `proof` not signed by solver. [LayerZero V2]

- [ ] **Fill deadline based on `block.timestamp` of destination chain assumed equal to origin**: Chains drift by seconds. A fill at destination's `block.timestamp == fillDeadline` is valid there but the origin sees it as PAST deadline when settling. Solver loses funds. Look for: any cross-chain timestamp comparison without slack. [Across]

- [ ] **Solver assumes destination calldata immutable, but bridge mutates**: Some bridges modify the calldata in transit (gas refund accounting, dst address mapping). Solver-side simulation against the original calldata doesn't match what destination actually receives. Look for: solver flows that don't re-simulate the destination calldata as the bridge will deliver it. [LayerZero V2]

## Cross-Chain Settlement Glue

- [ ] **Optimistic settlement without dispute window enforcement**: Settler accepts solver's claim, marks settled. Dispute window exists but the `settle()` function disburses funds BEFORE window elapses. Look for: any `disburseAfter` vs `disputeDeadline` gap. [Across; UMA]

- [ ] **Oracle attestation reusable across orderIds**: Oracle signs `(destinationChainId, fillHash)` but settler keys storage by `orderId`. Two orders that produce the same `fillHash` (e.g., identical Output struct) share one attestation. Look for: attestation domain not including orderId. [Across]

- [ ] **Finality assumption broken on reorg**: Solver fills on chain B at block N; chain B reorgs and the fill is gone, but solver's settlement claim already landed on chain A. Look for: lack of finality wait (block confirmations) before accepting attestations from probabilistic-finality chains (BSC, Polygon). [LayerZero V2 checklist]

- [ ] **`destinationChainId` is a curve-specific id, not EIP-155**: Some intent protocols use internal chain IDs (LayerZero, Wormhole) that aren't EIP-155 chain IDs. Mixing them up sends to wrong chain. Look for: chain ID mapping table integrity, especially after new chain adds. [LayerZero V2]

- [ ] **`callbackData` field forwards arbitrary call**: ERC-7683 implementations sometimes let the order include a `callbackData` to be executed on the user's behalf on destination after fill. If this is a generic `call`, it's a key-equivalent to the user. Look for: post-fill callbacks not strictly scoped (only specific selectors / targets / value caps). [Pashov]
