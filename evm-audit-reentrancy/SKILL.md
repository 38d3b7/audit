---
name: evm-audit-reentrancy
description: Reentrancy — beyond the basics. Read-only reentrancy across protocols, cross-function and cross-contract reentry, EIP-1153 transient-storage lock pitfalls, and callback-driven reentry via ERC721/1155/777 hooks. Load this for EVERY audit where any function calls out to a non-trusted address, transfers tokens with callbacks, or exposes a view function consumed by other protocols.
---

# EVM Audit — Reentrancy (Beyond `nonReentrant`)

Load this for **every** EVM contract that:
- Makes any external call (`.call`, `.transfer`, token transfer, callback)
- Mints/transfers `ERC721`, `ERC1155`, or `ERC777` (callback-bearing tokens)
- Exposes a view function (`getPrice`, `convertToAssets`, `getReserves`, `pendingRewards`, etc.) that other protocols may read
- Uses EIP-1153 transient storage (`tstore`/`tload`) for its reentrancy lock

## Why a dedicated skill

`nonReentrant` modifiers stop the most obvious case (single-function reentry against the same contract) and miss the rest:
- **Read-only reentrancy** doesn't write state, so no lock is even held — it tricks integrators who read your views mid-callback.
- **Cross-contract / cross-function reentry** routes through a *different* function or *different* contract that shares accounting.
- **Transient-storage locks** introduce a new class of bugs (forgetting to clear, nested-call leaks, wrong slot reuse).
- **Callback-bearing tokens** (ERC721 `onERC721Received`, ERC1155 `onERC1155Received`, ERC777 hooks) hand control to the recipient mid-state-update.

## Reference Files
- `references/checklist.md` — Full dense checklist
