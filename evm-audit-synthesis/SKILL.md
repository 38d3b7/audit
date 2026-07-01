---
name: evm-audit-synthesis
description: Finding verification and synthesis triage. Load ONLY during Phase 4 after all findings-*.md files exist. Runs every candidate finding through 13 verification gates to filter hallucinations, user-error false positives, governance-risk invalids, and severity miscalibrations before writing AUDIT-REPORT.md.
---

# EVM Audit — Synthesis & Verification Gates

**Do not load this skill during discovery (Phase 3).** Load it only when synthesizing findings into the final report.

## When to load

- All parallel domain agents have written their `findings-<skill>.md` files
- You are deduplicating, verifying, and writing `AUDIT-REPORT.md`
- You need to downgrade or reject findings that fail contest-grade triage

## Why a dedicated skill

Domain agents optimize for recall — they flag anything plausible. Synthesis optimizes for precision. Chain Shield's contest-hardened verification gates ([VERIFY_CHECKLIST.md](https://github.com/chain-shield/ai-agent-audit/blob/develop/VERIFY_CHECKLIST.md)) filter the most common false positives before they reach clients or GitHub issues.

## Workflow

1. Read compact headers from all `findings-*.md` first (severity counts + one-line title per finding); open full bodies only when dedup or verify-gates needs them — then deduplicate overlapping findings
2. For **each** candidate finding, walk through `references/verify-gates.md` in order
3. If any gate fails → mark **Invalid** or downgrade severity per the gate instructions
4. Run cross-cutting checks from `evm-audit-master/SKILL.md` Phase 4
5. Write `AUDIT-REPORT.md` with only gates-passed findings

Severity definitions live in `evm-audit-master/SKILL.md` — do not redefine them here.

## Reference Files

- `references/verify-gates.md` — Full 13-gate verification checklist (adapted from Chain Shield, MIT)
