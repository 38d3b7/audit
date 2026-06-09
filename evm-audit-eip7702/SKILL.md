---
name: evm-audit-eip7702
description: EIP-7702 "set EOA code" delegation (Pectra). Load when auditing wallets, delegation targets, sponsored-transaction flows, or any contract whose security model assumed `tx.origin == msg.sender` or that EOAs cannot execute code. Covers authorization replay, storage collisions in delegated code, sweeper/re-delegation attacks, init-on-every-tx pitfalls, and phishing via auth-tuple signatures.
---

# EVM Audit — EIP-7702 (Set-Code for EOAs)

Load this when the contract:
- **Is a 7702 delegation target** (the implementation contract pointed to by an EOA's `0xef0100||addr` code prefix)
- Verifies signatures from EOAs (now potentially executing code)
- Treats `tx.origin == msg.sender` as a "called by an EOA" check
- Uses raw ETH transfers (`.send`, `.transfer`, `.call{value:}`) to user-supplied addresses
- Implements a sponsored-transaction or relayer flow that uses 7702 auth tuples

## Why a dedicated skill

EIP-7702 (live with Pectra) lets an EOA temporarily delegate to a contract's bytecode for any number of transactions, via a signed `SetCodeTransaction` authorization tuple. This shipped a new class of footguns:

- **Authorization replay** — auth tuples are signed offchain with `(chainId, nonce, address)` and can be reused across chains if `chainId == 0`, or by anyone who picks up the signed tuple from the mempool.
- **Storage collisions** — delegated code reads/writes the EOA's storage. Standard libraries (OpenZeppelin Ownable, ReentrancyGuard) write to fixed slots; multiple delegated apps fighting over slot 0 is a real risk.
- **Re-delegation** — the EOA owner can re-delegate at any time and drain whatever the previous delegation left in storage or approvals.
- **`tx.origin == msg.sender` breaks** — that idiom no longer means "human"; an EOA may be running code.
- **Init-on-every-tx** — there is no constructor for the EOA's "implementation"; every entry must re-derive state safely or check `_initialized`.

## Reference Files
- `references/checklist.md` — Full dense checklist
