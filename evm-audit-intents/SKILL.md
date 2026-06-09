---
name: evm-audit-intents
description: Cross-chain intents — ERC-7683 and solver/filler-based settlement. Load when auditing an OriginSettler or DestinationSettler contract, an Across SpokePool-style flow, a CowSwap/UniswapX-derived intent flow, or any contract that lets a third-party "solver" fulfill a user's signed intent for a fee. Covers solver trust, settlement replay, origin/destination mismatch, fill-deadline assumptions, partial-fill accounting, and oracle-based settlement spoofing.
---

# EVM Audit — Cross-Chain Intents (ERC-7683)

Load this when the contract:
- Implements ERC-7683 `IOriginSettler` or `IDestinationSettler`
- Is a solver/filler contract that fulfills user orders for a fee
- Is a SpokePool-style cross-chain order escrow (Across, Synapse, deBridge DLN)
- Lets users sign a `GaslessCrossChainOrder` or equivalent intent struct
- Settles a fill from one chain by verifying proof / oracle on another chain

## Why a dedicated skill

ERC-7683 standardizes cross-chain intents: user signs an order on origin, solver fills on destination, origin settles the solver's fill claim. Each leg has its own attack surface:

- **Origin settler** holds user funds, releases them to whoever provides a valid fill claim — wrong proof check = drain
- **Destination settler / filler contract** disburses tokens to the user, must record fill atomically with payment
- **Solver economics** — front-runnable, sandwichable, or griefable solvers degrade UX or cause fund loss
- **Cross-chain glue** — message bridges, oracles, or attestations connecting the two; finality / replay assumptions break in reorgs

## Reference Files
- `references/checklist.md` — Full dense checklist
