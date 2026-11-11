```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens for the interactive raddle table change in `raddleModeRules`.

**Per change, the set I found and whether the diff covers it**

1. **Interactive mode text (`internal/loomengine/raddletable.go:52-61`)**
   - `contracts/stencils/raddle/raddle-template-agent.md` only injects `{{.mode_rules}}` at line 63. It says nothing about the table or its columns. Covered, nothing stale.
   - `contracts/specs/raddle-proposal-spec.md` describes the proposal file's `recipe`, `profile` and `depends_on` fields (lines 40-45). It does not describe the operator-facing table. Nothing stale.
   - `docs/overview.md`, `docs/skills.md`, `docs/reference/weaving-glossary.md` and the board skills mention raddle but none describes the interactive table. Nothing stale.
   - `PATTERN.md` and `pattern/PATTERN-stencil-ownership.md` only mention raddle in the context of stencil ownership. The diff touches no PATTERN entry, and the rubric's declared changes list is `none` for this round, so no declared entry is missing.
   - Test `internal/loomengine/raddletable_test.go:88,101` builds its expected text by calling `raddleModeRules` itself. The old wording is not hard-coded, so no assertion has gone stale. Covered.
   - The text still asks for the dependencies column by part slug, and the later "revise the recipes or profiles" instruction does not conflict with the new prose limit. Covered.

2. **Doc comment on `raddleModeRules` (`raddletable.go:41-48`)**
   - It matches the rendered interactive text: recipes, profiles and dependencies in the table, departures with their costs, drafts, and the prose limited to reasons and alternatives.
   - Raddle-table-wide wording about the comment's line breaks is the gate's job, not this lens's.

3. **Commit messages and departures**
   - The range has two commits, and neither carries a `Pattern-Departure:` trailer. None is needed.

4. **Change description (`_lym/landing/summary.md`)**
   - Its claims bear out against the diff: only the mode text changed, and the spec and parser are untouched. This is not my lens's subject beyond that check.

No findings.

**Tool calls**
1. `grep` over the raddle stencil and the proposal spec for table, column, recipe and profile wording. My context did not hold either file.
2. `grep` over the overview, skills, glossary and board skill docs for raddle lines mentioning tables, columns, recipes or profiles. My context did not hold those files.
