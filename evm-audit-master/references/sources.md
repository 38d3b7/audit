# EVM Audit Skill Suite — Source Tracking

Original baseline date fetched: 2026-02-27
v1.2.0 source refresh added: 2026-06-09

## Successfully Fetched Sources

### Dacian Articles (via archive.org — all 8 articles)
| Source | URL | Status |
|--------|-----|--------|
| DeFi Liquidation Vulnerabilities | https://dacian.me/defi-liquidation-vulnerabilities | ✅ via archive.org (50KB, truncated — very dense) |
| CLM Vulnerabilities | https://dacian.me/concentrated-liquidity-manager-vulnerabilities | ✅ via archive.org (20KB, complete) |
| DeFi Slippage Attacks | https://dacian.me/defi-slippage-attacks | ✅ via archive.org (27KB, complete) |
| Precision Loss Errors | https://dacian.me/precision-loss-errors | ✅ via archive.org (30KB, complete) |
| Signature Replay Attacks | https://dacian.me/signature-replay-attacks | ✅ via archive.org (15KB, complete) |
| DAO Governance DeFi Attacks | https://dacian.me/dao-governance-defi-attacks | ✅ via archive.org (50KB, truncated) |
| Inline Assembly Vulnerabilities | https://dacian.me/solidity-inline-assembly-vulnerabilities | ✅ via archive.org (35KB, complete) |
| Lending/Borrowing DeFi Attacks | https://dacian.me/lending-borrowing-defi-attacks | ✅ via archive.org (29KB, complete) |

### Devdacian GitHub
| Source | URL | Status |
|--------|-----|--------|
| AI Auditor Primers — base.primer.md | https://raw.githubusercontent.com/devdacian/ai-auditor-primers/main/primers/base.primer.md | ✅ (33KB) |
| AI Auditor Primers — amy.vault.erc4626.primer.md | https://raw.githubusercontent.com/devdacian/ai-auditor-primers/main/primers/amy.vault.erc4626.primer.md | ✅ listed (299KB — too large to process fully) |
| Primers repo tree | https://api.github.com/repos/devdacian/ai-auditor-primers/git/trees/main?recursive=1 | ✅ (2 primers found) |

### RareSkills
| Source | URL | Status |
|--------|-----|--------|
| Smart Contract Security | https://www.rareskills.io/post/smart-contract-security | ✅ via archive.org (50KB, truncated) |
| UUPS Proxy | https://www.rareskills.io/post/uups-proxy | ✅ via archive.org (21KB, complete) |

### Sigma Prime
| Source | URL | Status |
|--------|-----|--------|
| Governance & DAOs | https://blog.sigmaprime.io/governance-dao.html | ✅ direct (17KB) |
| Oracles & Pricing | https://blog.sigmaprime.io/oracles-and-pricing.html | ✅ direct (20KB) |
| Liquid Restaking | https://blog.sigmaprime.io/liquid-restaking.html | ✅ direct (26KB) |

### Cyfrin / Dacian
| Source | URL | Status |
|--------|-----|--------|
| Chainlink Oracle Security | https://medium.com/cyfrin/chainlink-oracle-defi-attacks-93b6cb6541bf | ✅ direct (22KB) |

### SWC Registry
| Source | URL | Status |
|--------|-----|--------|
| README | https://raw.githubusercontent.com/SmartContractSecurity/SWC-registry/master/README.md | ✅ (registry no longer maintained since 2020, superseded by EEA EthTrust Security Levels) |

## Failed Sources

| Source | URL | Reason |
|--------|-----|--------|
| Dacian direct (all URLs) | https://dacian.me/* | 429 — Vercel Security Checkpoint (bot protection). Resolved via archive.org. |
| ConsenSys Best Practices | https://consensys.github.io/smart-contract-best-practices/attacks/ | 404 — GitHub Pages site no longer at this path |
| MixBytes — Liquid Staking | https://mixbytes.io/blog/liquid | Fetch failed (connection error) |
| MixBytes — Account Abstraction | https://mixbytes.io/blog/account-abstraction | Not attempted (similar expected failure) |
| Solodit Checklist | https://solodit.cyfrin.io/checklist | JS-rendered, requires browser. Archive.org not attempted. |
| Blast Integration Bugs | https://nirlin-blast-bugs.notion.site/... | Notion pages require JS rendering |
| DeFi Llama Hacks | https://defillama.com/hacks | JS-rendered dashboard |
| Rekt News | https://rekt.news | Referenced in articles but not independently fetched |
| OpenZeppelin CHANGELOG | https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/CHANGELOG.md | Not fetched (very large, low signal-to-noise ratio) |

## v1.2.0 Source Refresh (added 2026-06-09)

The following sources were folded into the new domain skills (`evm-audit-reentrancy`, `evm-audit-eip7702`, `evm-audit-modular-accounts`, `evm-audit-intents`, `evm-audit-mev`) and cross-referenced into existing skills.

### Spec Sources (canonical — fetched from EIP/ERC documents and reference implementations)
| Source | URL | Used For |
|--------|-----|----------|
| EIP-7702 | https://eips.ethereum.org/EIPS/eip-7702 | `evm-audit-eip7702` |
| EIP-1153 (Transient Storage) | https://eips.ethereum.org/EIPS/eip-1153 | `evm-audit-reentrancy` |
| EIP-6780 (SELFDESTRUCT change) | https://eips.ethereum.org/EIPS/eip-6780 | `evm-audit-eip7702` |
| ERC-7579 (Modular Smart Account) | https://eips.ethereum.org/EIPS/eip-7579 | `evm-audit-modular-accounts` |
| ERC-6900 (Modular Account) | https://eips.ethereum.org/EIPS/eip-6900 | `evm-audit-modular-accounts` |
| ERC-7484 (Registry Adapter) | https://eips.ethereum.org/EIPS/eip-7484 | `evm-audit-modular-accounts` |
| ERC-7683 (Cross-Chain Intents) | https://eips.ethereum.org/EIPS/eip-7683 | `evm-audit-intents` |
| ERC-7201 (Namespaced Storage) | https://eips.ethereum.org/EIPS/eip-7201 | `evm-audit-eip7702`, `evm-audit-modular-accounts` |
| ERC-1271 (Contract Signatures) | https://eips.ethereum.org/EIPS/eip-1271 | `evm-audit-eip7702` |
| ERC-4337 v0.7 | https://eips.ethereum.org/EIPS/eip-4337 | `evm-audit-modular-accounts`, `evm-audit-mev` |

### Aggregated Findings Databases
| Source | URL | Notes |
|--------|-----|-------|
| Solodit | https://solodit.cyfrin.io/ | Aggregator of public audit findings (Code4rena, Sherlock, Cantina, Spearbit, ToB, Trail of Bits). Cited as `[Solodit]` when a finding is best evidenced there. |
| Code4rena Reports | https://code4rena.com/reports | Public contest reports — cited `[C4]`. |
| Sherlock Contests | https://audits.sherlock.xyz/contests | Cited `[Sherlock]`. |
| Cantina Competitions | https://cantina.xyz/competitions | Cited `[Cantina]`. |
| Pashov Audit Group | https://github.com/pashov/audits | Cited `[Pashov]`. |
| Immunefi Explore | https://immunefi.com/explore/ | Disclosed bug bounty reports — cited `[Immunefi]`. |

### Auditor / Curriculum Sources
| Source | URL | Notes |
|--------|-----|-------|
| Trail of Bits — Building Secure Contracts | https://github.com/crytic/building-secure-contracts | Cited `[ToB]`. Also covers Slither detectors. |
| OpenZeppelin Contracts 5.x | https://github.com/OpenZeppelin/openzeppelin-contracts/releases | Cited `[OZ 5.x]`. Used for `ReentrancyGuardTransient`, Initializable behavior, ERC-7201 patterns. |
| Cyfrin Updraft | https://updraft.cyfrin.io/ | Cited `[Cyfrin Updraft]`. |
| Flashbots Research | https://writings.flashbots.net/ | Cited `[Flashbots]`. Used for MEV / sandwich / JIT / builder trust. |

### Reference Implementations Consulted
- **Rhinestone ModuleKit** (ERC-7579 reference) — module install patterns, registry adapters
- **Biconomy Nexus** — session-key validators and nonce-key namespacing
- **Across Protocol** — ERC-7683 reference settlers, optimistic oracle dispute window
- **Uniswap X** — Dutch-auction intent / RFQ / exclusivity window patterns
- **Permit2** — witness binding for batch transfer with calldata commitments

### Notes on Stale / Replaced Sources

- **SWC Registry** has not been updated since 2020 and has been superseded by the [EEA EthTrust Security Levels Specification](https://entethalliance.org/specs/ethtrust-sl/). All SWC weaknesses were incorporated into that spec. Existing checklist items already cover the most critical SWC entries.
- **Devdacian's base.primer.md** (33KB) is an extremely high-quality comprehensive primer covering lending, liquidation, signatures, precision, slippage, oracle, CLM, staking, auction, and reentrancy vulnerability patterns with detailed invariants and checklists. Content from this primer has been cross-referenced and integrated.
- All Dacian articles were successfully fetched via Wayback Machine (archive.org) after direct access was blocked by Vercel bot protection.
- Total extracted content (v1.0.0 baseline): ~450KB of high-quality audit security content processed into checklist items across 8+ skill files.
- v1.2.0 added ~110 new checklist items across 5 new domains, sourced predominantly from post-Feb-2026 findings databases (Solodit, C4, Sherlock, Cantina, Pashov) and the EIP/ERC spec documents listed above.
