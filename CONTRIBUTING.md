# Contributing

Thanks for helping keep this audit suite sharp. The goal is **dense, non-obvious, sourced checklist items** that catch bugs basic tools miss — not generic best-practice lists.

---

## Quick Rules

1. **Don't add basics.** "Use SafeERC20", "follow CEI", "don't use `tx.origin`" — already known by every junior auditor. Items must be **non-obvious** or **recent**.
2. **Always cite a source.** Use the existing tag set in [`evm-audit-master/SKILL.md`](./evm-audit-master/SKILL.md#source-attribution-key) (e.g. `[Solodit]`, `[C4]`, `[Pashov]`, `[EIP-7683]`). Open a PR to add a new tag if needed.
3. **Include a `Look for:` hint.** Tell the auditor exactly what code shape to grep / which AST pattern to scan. Without this, items are theoretical.
4. **Each item is a single `[ ]` line.** Title in **bold**, then description, then `Look for:`, then `[source]` tag.

---

## File Layout

Every skill folder mirrors the same structure:

```
evm-audit-<domain>/
  SKILL.md                  # frontmatter + brief overview (when to load)
  references/
    checklist.md            # the dense [ ] items
```

The root [`SKILL.md`](./SKILL.md) is the agent entry point (drop-in replacement for the `audit` plugin skill). The root [`README.md`](./README.md) is for humans browsing the repo on GitHub.

---

## Adding a New Domain Skill

1. **Create the folder + files**:
   ```
   evm-audit-<domain>/SKILL.md
   evm-audit-<domain>/references/checklist.md
   ```

2. **`SKILL.md`** — keep it under 30 lines:
   ```markdown
   ---
   name: evm-audit-<domain>
   description: <one-sentence purpose, used by the skill router>
   ---

   # EVM Audit — <Domain>

   Load this when the contract:
   - <criterion 1>
   - <criterion 2>

   ## Why a dedicated skill
   <2-4 sentences on what's special about this domain>

   ## Reference Files
   - `references/checklist.md` — Full dense checklist
   ```

3. **`references/checklist.md`** — group items under `## Section` headings. Each item:
   ```markdown
   - [ ] **<Bold title — name the pattern>**: <One or two sentences on the bug, why it matters, what makes it non-obvious>. Look for: <exact code shape or grep target>. [source]
   ```

4. **Register the skill** in three places:
   - [`evm-audit-master/SKILL.md`](./evm-audit-master/SKILL.md) — add a row to the **All Skills** table and the **Routing Table**.
   - [`README.md`](./README.md) — add a row to the **Skills** table.
   - [`SKILL.md`](./SKILL.md) (root) — add a row to **Skills Available**.

5. **Bump the version** in `CHANGELOG.md` and `evm-audit-master/SKILL.md` (header line). Use semver: new domain = minor bump, content-only refresh = patch.

6. **Open a PR.** Title format: `feat(<domain>): add evm-audit-<domain> (N items)`.

---

## Adding Items to an Existing Skill

- Find the right `## Section` heading (add a new section if it's a distinct sub-domain).
- Keep items terse. If you need a paragraph, the item is probably two items.
- Update the item count in [`evm-audit-master/SKILL.md`](./evm-audit-master/SKILL.md) if it changes by 5+.

---

## Adding a New Source

If you're citing something not in the [Source Attribution Key](./evm-audit-master/SKILL.md#source-attribution-key):

1. Add a short tag (e.g. `[Halborn]`, `[OpenZeppelin Defender]`) to the key in `evm-audit-master/SKILL.md`.
2. Add a row to `evm-audit-master/references/sources.md` with the URL and what you used it for.
3. Add the source to `README.md` attribution list if it's a significant new feed.

---

## Don'ts

- **Don't restate the standard finding format inside skills** — it's defined once in `evm-audit-master/SKILL.md`.
- **Don't add severity scoring inside checklist items** — severity is determined per-finding during synthesis.
- **Don't add tutorial / explanatory content** to `references/checklist.md` — that goes in `SKILL.md` if anywhere.
- **Don't reorder existing items** without reason — preserves PR review diff sanity.

---

## Review Bar

Items in a PR are accepted when:

- Non-obvious (an experienced auditor would have to think to catch it)
- Sourced (cites a finding, spec, or article)
- Actionable (`Look for:` lets you scan code for it)
- Specific (names a function pattern, not "be careful with X")

Generic OWASP-style items, things already covered by Slither defaults, or pure stylistic advice will be rejected.
