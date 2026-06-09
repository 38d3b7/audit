# Changelog

All notable changes to this audit skill suite. Follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) loosely; semantic versioning where: **major** = breaking layout/format change, **minor** = new domain skill, **patch** = checklist content updates within existing skills.

## [1.2.0] — 2026-06-09

Team-vendored release. Forked from `austintgriffith/evm-audit-skills` v1.0.0 (Feb-2026 baseline), extended for in-house team use.

### Added

- **`evm-audit-reentrancy/`** — New domain. Read-only reentrancy (Curve-style integrator drains), cross-function & cross-contract reentry, EIP-1153 transient-storage lock pitfalls, ERC721/1155/777 callback reentry, EIP-7702 EOA-callback reentry. (~28 items)
- **`evm-audit-eip7702/`** — New domain. Pectra EOA delegation: auth tuple replay (`chainId == 0`), storage-slot collisions in delegated code, re-delegation / sweeper drains, broken `tx.origin == msg.sender` idioms, native-ETH-to-EOA now triggers code, sponsored-tx hijacks. (~22 items)
- **`evm-audit-modular-accounts/`** — New domain. ERC-7579 / ERC-6900 / ERC-7484: malicious validator/executor/hook/fallback modules, install authorization, registry trust, fallback-handler hijack of `isValidSignature`, session-key scope binding. (~20 items)
- **`evm-audit-intents/`** — New domain. ERC-7683 cross-chain intents: order-hash chain-id binding, solver replay across bridges, partial-fill accounting, optimistic settlement dispute windows, post-fill callback scoping. (~18 items)
- **`evm-audit-mev/`** — New domain. Sandwich/JIT, missing `deadline`/`minOut`, frontrun-mint via block-field randomness, atomic-arb griefing, RFQ quote staleness, bundler/builder trust, cross-domain MEV (L2 sequencers, intent solvers). (~22 items)
- **Refreshed sources.** Added Solodit, Code4rena, Sherlock, Cantina, Pashov Audit Group, Trail of Bits (Building Secure Contracts + Slither), OpenZeppelin Contracts 5.x, Cyfrin Updraft, Immunefi disclosed reports, Flashbots research, and canonical EIP/ERC specs for new domains.
- **`CONTRIBUTING.md`** — Single-page contributor guide.
- **Root `SKILL.md`** — Drop-in replacement for the upstream `audit` plugin skill that points at the local master index.

### Changed

- **`evm-audit-master/SKILL.md`** — Updated to 25-skill index with refreshed routing table. Always-on baseline expanded from 2 → 4 skills (`general` + `precision-math` + `reentrancy` + `access-control`).
- **`evm-audit-master/references/sources.md`** — Added v1.2.0 refresh section tracking new spec / aggregator / curriculum sources.
- **`README.md`** — Replaced upstream's contributor-facing README with team-vendored version; full attribution preserved.

### Removed

- Runtime dependency on the external `austintgriffith/evm-audit-skills` GitHub URLs. All checklists are now vendored and resolved locally.

---

## [1.0.0] — 2026-02-28 (upstream baseline)

Initial commit of [`austintgriffith/evm-audit-skills`](https://github.com/austintgriffith/evm-audit-skills): 20 EVM audit skills, ~1900 lines of security checklists across master, general, precision-math, ERC20, DeFi AMM/lending/staking, ERC4626, ERC4337, bridges, proxies, signatures, governance, oracles, assembly, chain-specific, flashloans, ERC721, DoS, and access control.

Full attribution preserved verbatim in each `evm-audit-*` folder and in [`evm-audit-master/SKILL.md`](./evm-audit-master/SKILL.md#source-attribution-key).
