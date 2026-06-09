# Audit — EVM Smart Contract Security Audit Skills

A suite of 24 agent skills for deep EVM smart contract security audits, designed to run as parallel sub-agents against domain-specific checklists.

**~2,400 lines of checklist content. 650+ individual findings. 24 specialized domains.**

This repository is the team-vendored, extended successor to [`austintgriffith/evm-audit-skills`](https://github.com/austintgriffith/evm-audit-skills) (Feb 2026 baseline). It adds five new domain skills, refreshes source attributions with current findings databases, and removes the runtime dependency on a third-party GitHub account.

---

## Quick Start

```bash
# From an agent session, point your audit driver at the master index:
./evm-audit-master/SKILL.md
```

The top-level [`SKILL.md`](./SKILL.md) is a drop-in replacement for the `audit` plugin skill — it routes the agent into `evm-audit-master/`, which contains the routing table and methodology.

---

## Skills

| # | Skill | What It Covers |
|---|-------|---------------|
| 1 | [`evm-audit-master`](./evm-audit-master/) | **Start here.** Routing table, methodology, standard finding format |
| 2 | [`evm-audit-general`](./evm-audit-general/) | Cross-cutting EVM footguns — applies to every contract |
| 3 | [`evm-audit-precision-math`](./evm-audit-precision-math/) | Division ordering, rounding direction, downcast overflow, decimal mismatches |
| 4 | [`evm-audit-reentrancy`](./evm-audit-reentrancy/) | **NEW.** Read-only reentrancy, transient-storage (EIP-1153) lock bugs, callback reentry |
| 5 | [`evm-audit-access-control`](./evm-audit-access-control/) | Ownership, roles, 2-step transfer, emergency controls |
| 6 | [`evm-audit-erc20`](./evm-audit-erc20/) | Fee-on-transfer, rebasing, ERC777 hooks, approve races, weird tokens |
| 7 | [`evm-audit-defi-amm`](./evm-audit-defi-amm/) | Uniswap V3/V4, slippage attacks, CLM vulnerabilities, TWAP manipulation |
| 8 | [`evm-audit-defi-lending`](./evm-audit-defi-lending/) | Liquidation patterns, bad debt, collateral hiding, non-18 decimal failures |
| 9 | [`evm-audit-defi-staking`](./evm-audit-defi-staking/) | Liquid staking, restaking, EigenLayer, cooldown exploitation |
| 10 | [`evm-audit-erc4626`](./evm-audit-erc4626/) | Vault share math, inflation attack, rounding direction, 85+ patterns |
| 11 | [`evm-audit-erc4337`](./evm-audit-erc4337/) | Account abstraction, paymasters, session keys, bundler trust |
| 12 | [`evm-audit-eip7702`](./evm-audit-eip7702/) | **NEW.** EOA delegation (Pectra), set-code, re-delegation, sweeper attacks |
| 13 | [`evm-audit-modular-accounts`](./evm-audit-modular-accounts/) | **NEW.** ERC-7579 / ERC-6900 / ERC-7484 modular smart accounts |
| 14 | [`evm-audit-bridges`](./evm-audit-bridges/) | LayerZero V2, CCIP, Wormhole, Across, message replay, finality |
| 15 | [`evm-audit-intents`](./evm-audit-intents/) | **NEW.** ERC-7683 cross-chain intents, solver/filler trust |
| 16 | [`evm-audit-proxies`](./evm-audit-proxies/) | UUPS, Transparent, Beacon, Diamond, storage collisions, initializer bugs |
| 17 | [`evm-audit-signatures`](./evm-audit-signatures/) | Replay attacks, ecrecover, EIP-712, permit edge cases, malleability |
| 18 | [`evm-audit-governance`](./evm-audit-governance/) | Flash loan voting, totalPower manipulation, proposal ordering, fake CREATE2 proposals |
| 19 | [`evm-audit-oracles`](./evm-audit-oracles/) | Chainlink staleness, minAnswer/maxAnswer, L2 sequencer, TWAP limits |
| 20 | [`evm-audit-assembly`](./evm-audit-assembly/) | Memory corruption, FMPA bugs, non-existent contract calls, uint128 overflow |
| 21 | [`evm-audit-chain-specific`](./evm-audit-chain-specific/) | Arbitrum, Optimism, zkSync, Blast, BSC — L2 quirks |
| 22 | [`evm-audit-flashloans`](./evm-audit-flashloans/) | Flash loan attack patterns, oracle manipulation, governance exploits |
| 23 | [`evm-audit-mev`](./evm-audit-mev/) | **NEW.** Sandwich, JIT liquidity, missing deadline/minOut, commit-reveal gaps |
| 24 | [`evm-audit-erc721`](./evm-audit-erc721/) | NFT callbacks, enumeration DoS, royalty bypass, wrapped collections |
| 25 | [`evm-audit-dos`](./evm-audit-dos/) | Denial of service: unbounded loops, gas griefing, return-data bombs |

Each skill has the same structure:

```
evm-audit-<domain>/
  SKILL.md                  # frontmatter (name, description) + brief overview
  references/checklist.md   # dense checklist of `[ ]` items with source tags
```

---

## What's New vs Upstream

Versus the Feb-2026 `austintgriffith/evm-audit-skills` baseline:

**5 new domain skills:**
- `evm-audit-reentrancy` — promoted from a footnote in `general` into its own domain, with read-only reentrancy, cross-function/cross-contract reentry, EIP-1153 transient-storage lock pitfalls, and ERC721/1155/777 callback reentry.
- `evm-audit-eip7702` — EOA delegation under Pectra: storage collisions in delegated code, broken `tx.origin == msg.sender` assumptions, re-delegation/sweeper attacks, init-on-every-tx, phishing via set-code, relayer trust.
- `evm-audit-modular-accounts` — ERC-7579 / ERC-6900 / ERC-7484: malicious validator/executor modules, hook ordering, install auth, registry trust, fallback-handler hijack.
- `evm-audit-intents` — ERC-7683 cross-chain intents: solver/filler trust, settlement replay, origin/destination mismatch, fill-deadline assumptions, partial-fill accounting.
- `evm-audit-mev` — sandwich/JIT, missing `deadline`/`minOut`, private-mempool assumptions, backrunnable state, mint frontrunning, commit-reveal gaps.

**Refreshed source attribution** (see Attribution section below): added Solodit, Code4rena / Sherlock / Cantina, Pashov Audit Group, Trail of Bits, OpenZeppelin Contracts 5.x, Cyfrin Updraft, and Immunefi disclosed reports.

---

## How to Run an Audit

See [`evm-audit-master/SKILL.md`](./evm-audit-master/SKILL.md) for the full methodology. Short version:

1. **Recon** — fetch contracts, map inheritance, identify entry points
2. **Skill selection** — load `general` + `precision-math` + `reentrancy` + `access-control` always, then route based on contract surface
3. **Parallel agents** — one opus sub-agent per skill, each writes `findings-<skill>.md`
4. **Synthesize** — dedupe, look for cross-cutting concerns, produce `AUDIT-REPORT.md`
5. **File issues** — `gh issue create` for Medium+ findings

---

## Attribution

Originally derived from **[`austintgriffith/evm-audit-skills`](https://github.com/austintgriffith/evm-audit-skills)** — the Feb 2026 baseline of 20 skills. Huge thanks to that work; all upstream content is preserved verbatim in the corresponding skill folders.

Source material across all skills:

- **[Dacian](https://dacian.me)** — 8 deep-dive security articles (liquidation, CLM, slippage, precision, signatures, governance, assembly, lending)
- **[beirao.xyz](https://beirao.xyz)** — audit checklist
- **[devdacian/ai-auditor-primers](https://github.com/devdacian/ai-auditor-primers)** — `base.primer.md`
- **[Sigma Prime](https://blog.sigmaprime.io/)** — governance, oracles, liquid restaking
- **[RareSkills](https://www.rareskills.io/)** — smart contract security, UUPS proxy
- **[Decurity](https://github.com/Decurity)** — AMM/CDP/LSD protocol-specific checklists
- **[d-xo/weird-erc20](https://github.com/d-xo/weird-erc20)** — token weirdness catalog
- **[Spearbit](https://spearbit.com/)** — bridge security checklist
- **[Hacken](https://hacken.io/)** — Uniswap V4 hooks audit guide
- **[Cyfrin / Cyfrin Updraft](https://www.cyfrin.io/)** — Chainlink oracle security, security curriculum
- **[OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts)** — Contracts 5.x migration notes & security advisories
- **[Trail of Bits](https://github.com/crytic/building-secure-contracts)** — Building Secure Contracts, Slither detectors
- **[Solodit](https://solodit.cyfrin.io/)** — aggregated public audit findings database
- **[Code4rena](https://code4rena.com/reports)** — public contest reports
- **[Sherlock](https://audits.sherlock.xyz/contests)** — public contest reports
- **[Cantina](https://cantina.xyz/competitions)** — public competition reports
- **[Pashov Audit Group](https://github.com/pashov/audits)** — published audit reports
- **[Immunefi](https://immunefi.com/explore/)** — disclosed bug bounty reports
- **[multichain-auditor](https://github.com/0xJuancito/multichain-auditor)** — 0xJuancito's L2/multichain checklist
- **[mixbytes CREATE2](https://mixbytes.io/blog/collisions-of-create2-addresses)** — CREATE2 security
- **[SWC Registry](https://swcregistry.io/)** — Smart Contract Weakness Classification
- **[Vectorized / EIP-7702 community notes](https://eips.ethereum.org/EIPS/eip-7702)** — Pectra delegation patterns
- **[ERC-7579](https://eips.ethereum.org/EIPS/eip-7579)** / **[ERC-6900](https://eips.ethereum.org/EIPS/eip-6900)** / **[ERC-7484](https://eips.ethereum.org/EIPS/eip-7484)** — modular account standards
- **[ERC-7683](https://eips.ethereum.org/EIPS/eip-7683)** — cross-chain intents
- **[LayerZero V2](https://docs.layerzero.network/v2)** / **[CCIP](https://docs.chain.link/ccip)** / **[Wormhole](https://docs.wormhole.com/)** / **[Across](https://docs.across.to/)** — bridge integration security
- **[Arbitrum](https://docs.arbitrum.io/)** / **[Blast](https://docs.blast.io/)** — official L2 docs

---

## Contributing — Adding a Domain

To add a new audit domain:

1. Create the folder:
   ```
   evm-audit-<domain>/
     SKILL.md
     references/checklist.md
   ```

2. **`SKILL.md`** — start with YAML frontmatter:
   ```
   ---
   name: evm-audit-<domain>
   description: <one-sentence description for the agent's skill router>
   ---
   ```
   Keep the body short (under 30 lines): say when to load, and point at `references/checklist.md`.

3. **`references/checklist.md`** — dense `[ ]` items. Each item:
   - **Bolded title** describing the pattern.
   - A `Look for:` hint telling the auditor what to grep / which code shape.
   - A `[source]` tag (see Attribution above for the canonical short-codes).
   - Excludes basics already covered by other domains.

4. **Register the skill** in:
   - `evm-audit-master/SKILL.md` — add to the routing table and the index.
   - `README.md` — add to the Skills table.
   - `SKILL.md` (root) — add to the Skills Available table.

5. Open a PR. Severity definitions and finding format are defined once in `evm-audit-master/SKILL.md`; don't re-define them per skill.

---

## Version

`v1.2.0` — derived from `austintgriffith/evm-audit-skills` Feb-2026 baseline (`v1.0.0`), extended with 5 new domains and refreshed sources.

## License

Inherits the upstream project's license — see [`austintgriffith/evm-audit-skills`](https://github.com/austintgriffith/evm-audit-skills). Add an explicit `LICENSE` file before publishing.
