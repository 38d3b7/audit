---
name: evm-audit-mev
description: MEV — sandwich, JIT liquidity, frontrun, backrun, and bundler/builder trust assumptions. Load when auditing any contract whose state changes are visible in the public mempool and are economically valuable to reorder, including swaps, mints, liquidations, auctions, governance flips, and oracle-touching state writes.
---

# EVM Audit — MEV (Maximal Extractable Value)

Load this when the contract:
- Lets users **swap, mint, or claim** in the public mempool
- Has any function whose **success depends on a price or quantity** the attacker can move (`minOut`, `maxIn`, `expectedShares`)
- Mints NFTs with rarity / commit-reveal
- Runs auctions / bidding flows
- Calls **oracles whose update transactions are observable in the mempool**
- Issues a **bundle / userOp / intent** that a builder or bundler can reorder

## Why a dedicated skill

The existing checklists touch MEV (slippage in `defi-amm`, governance flash-loan voting, oracle manipulation), but never gather it as a domain. MEV-adjacent bugs are the single largest category of "loss after launch" in production DeFi:

- **Sandwich** — attacker frontruns user's swap, then backruns to capture slippage
- **JIT** — bot adds liquidity right before a known swap, removes after
- **Frontrun-mint** — attacker reads pending mint, mints first, captures rarity / allowlist
- **Backrun-arbitrage griefing** — atomic backrun reverts after taking benefit, leaving user out of gas
- **Bundler / builder trust** — private mempools, RFQ systems, intent solvers may collude

## Reference Files
- `references/checklist.md` — Full dense checklist
