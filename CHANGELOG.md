# Changelog

All notable changes to this audit skill suite. Follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) loosely; semantic versioning where: **major** = breaking layout/format change, **minor** = new domain skill, **patch** = checklist content updates within existing skills.

## [1.4.0] — 2026-07-01

### Changed

- **`evm-audit-master/SKILL.md`** — Context-scoped sub-agents + summary-first synthesis for token efficiency (Phase 3 scoped source packages + required findings-header-shape bullet; Phase 4 header-first fan-in).
- **Plugin manifest** — v1.4.0.

---

## [1.3.0] — 2026-06-09

Chain Shield port. Adapted portable knowledge from [chain-shield/ai-agent-audit](https://github.com/chain-shield/ai-agent-audit) (MIT, `develop` branch).

### Added

- **`evm-audit-synthesis/`** — Phase 4 only. 13 verification gates adapted from Chain Shield VERIFY_CHECKLIST.md: pre-gate sanity (hallucination filter), scope, user-error, impact/likelihood matrix, governance-risk, speculation, documentation-only, by-design, safeguards, cross-cutting synthesis checks.
- **`evm-audit-game-theory/`** — ~24 items: keeper/liveness economics, queue-order MEV, griefing economics, governance capture, actor-oriented review, six-class invariant hunting (Arithmetic, Balance, Permission, Referential, State Machine, Temporal). Patterns from Chain Shield `threat_models/patterns.rs`.
- **Cherry-picked Chain Shield patterns** into existing skills: governance (+2), general (+3), oracles (+2), proxies (+1), intents (+2), defi-lending (+2).
- **README "Optional Automation Layer"** — documents Chain Shield as complementary execution-layer tool.

### Changed

- **`evm-audit-master/SKILL.md`** — 27-skill index; Phase 4 now requires dedicated synthesis agent with `evm-audit-synthesis`; routing table adds game-theory.
- **Plugin manifest** — v1.3.0, registers synthesis + game-theory skills.

### Attribution

- Chain Shield (MIT): VERIFY_CHECKLIST.md → `evm-audit-synthesis/references/verify-gates.md`; game-theory patterns → `evm-audit-game-theory/references/checklist.md`.

---

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
