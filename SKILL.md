---
name: audit
description: Deep EVM smart contract security audit system. Use when asked to audit a contract, find vulnerabilities, review code for security issues, or file security issues on a GitHub repo. Covers 650+ non-obvious checklist items across 24 domains via parallel sub-agents. Different from the security skill (which teaches defensive coding) — this is for systematically auditing contracts you didn't write.
---

# EVM Smart Contract Audit

A full audit system for any EVM contract. Runs parallel specialist agents against domain-specific checklists, synthesizes findings, and files GitHub issues.

This is the **team-vendored** version of the audit suite. All checklists live in this repository — no live fetches from third-party accounts.

## The Checklists

24 specialized skills covering every major vulnerability domain. Start with the master index:

```
./evm-audit-master/SKILL.md
```

The master index contains:
- Full routing table (which skills to load for which contract types)
- The complete audit methodology (recon → parallel agents → synthesis → issues)
- Standard finding format with severity definitions

Each skill checklist lives at:
```
./<skill-name>/references/checklist.md
```

## Skills Available

| Skill | When to Load |
|-------|-------------|
| `evm-audit-general` | Always |
| `evm-audit-precision-math` | Always |
| `evm-audit-reentrancy` | Always (read-only + transient storage) |
| `evm-audit-access-control` | Always |
| `evm-audit-erc20` | Contract interacts with ERC20 tokens |
| `evm-audit-defi-amm` | AMM, DEX, Uniswap V3/V4, liquidity pools |
| `evm-audit-defi-lending` | Lending, borrowing, CDP, liquidations |
| `evm-audit-defi-staking` | Staking, liquid staking, restaking, EigenLayer |
| `evm-audit-erc4626` | Vaults, share/asset conversion |
| `evm-audit-erc4337` | Account abstraction, paymasters, session keys |
| `evm-audit-eip7702` | EOA delegation (Pectra), set-code authorizations |
| `evm-audit-modular-accounts` | ERC-7579 / ERC-6900 / ERC-7484 modular smart accounts |
| `evm-audit-bridges` | Cross-chain, LayerZero, CCIP, Wormhole |
| `evm-audit-intents` | ERC-7683 cross-chain intents, solver/filler systems |
| `evm-audit-proxies` | Upgradeable contracts, UUPS, Transparent, Diamond |
| `evm-audit-signatures` | Off-chain signatures, EIP-712, permits |
| `evm-audit-governance` | DAO voting, timelocks, multi-sig |
| `evm-audit-oracles` | Chainlink, TWAP, Pyth, price feeds |
| `evm-audit-assembly` | Inline assembly, Yul, CREATE2 |
| `evm-audit-chain-specific` | Non-mainnet: Arbitrum, OP, zkSync, Blast, BSC |
| `evm-audit-flashloans` | Flash loan attack vectors |
| `evm-audit-mev` | Sandwich, JIT liquidity, frontrun-sensitive flows |
| `evm-audit-erc721` | NFTs, ERC721, ERC1155 |
| `evm-audit-dos` | DoS, unbounded loops, gas griefing |

## How To Run An Audit

1. Read `evm-audit-master/SKILL.md` — it has the full pipeline
2. Read the contract(s)
3. Select 5-8 skills using the routing table
4. Spawn one opus sub-agent per skill (parallel)
5. Each agent walks its checklist and writes `findings-<skill>.md`
6. Synthesize all findings into `AUDIT-REPORT.md`
7. File GitHub issues for Medium severity and above

## Invocation

```
Audit this contract and file issues: https://github.com/owner/repo/blob/main/contracts/Foo.sol
Checklists: <this-repo>/evm-audit-master/SKILL.md
```

## Sources

Built from research by Dacian, beirao.xyz, Sigma Prime, RareSkills, Decurity, weird-erc20, Spearbit, Hacken, OpenZeppelin, Cyfrin, Trail of Bits, Pashov Audit Group, Solodit, Code4rena, Sherlock, Cantina, Immunefi disclosed reports, and more. See `README.md` for full attribution.

Originally derived from [`austintgriffith/evm-audit-skills`](https://github.com/austintgriffith/evm-audit-skills) (Feb 2026 baseline), vendored and extended for team use.
