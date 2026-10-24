```yaml
lens: code-affected-set
outcome: reported
```

**Task: list what the diff forces outside its own lines, per change, from the search results.**

Affected set per change:

1. **New check `pattern-card-headroom`** (`patternplan.go`)
   - **Docs:** `internal/loomshed/doc.go:24-33` and `contracts/specs/loom-plan-spec.md:409-420` state the check, the "four" count and the way forward. Covered.
   - **Stencil:** `loom-template-plan.md:101` names the check. Covered.
   - **Tests:** `gates_test.go` counts the check and its `Card` per row. Covered.
   - **Spec line 547:** it names only `pattern-headroom` and `pattern-line-unprescribed` as checking the prescribed lines. Not wrong, since it is an example of two checks.
   - **Other consumers:** a repo-wide grep finds no other enumerator of the check names, such as a stencil, a recipe or a CLI table.
   - **Parity test:** `internal/loomcli/parity_pattern_integration_test.go:76` asserts that `pattern-headroom` appears in both the gate and verb outputs. Both outputs still carry it, and the added finding does not disturb a substring match. Covered.

2. **`patternHeadroomWayForward` constant extracted from the `pattern-headroom` message**
   - The message text is unchanged.
   - No test or doc quotes the old inline string beyond the `lyx loom decision add` fragment, which the constant still holds. Covered.

3. **New `planparser.CardNamesPatternEntry`**
   - Its only caller is `patternplan.go:129`.
   - It has a test in `patternentry_test.go`.
   - The `Card.Text` comment in `plan.go` was updated to match. Covered.
   - The change keeps card text going through `planparser`, as `PATTERN-sole-parsers` requires.

4. **Stencil `loom-template-plan.md`**
   - The diff only adds two lines to the PATTERN-prescription passage and drops or weakens no existing rule.
   - The added rule that a removing card names the ID in backticks matches the crediting rule in the decision record. Covered.

5. **`PATTERN.md` and `pattern/`**
   - The diff declares no PATTERN change, and the diff stat shows no change to either, so nothing is forced there.

Findings: none.

Tool calls:
1. Grep for all four check names repo-wide, to find every enumerator, doc, test and stencil. My context held only the diff.
2. Grep for `CardNamesPatternEntry` and `CardPatternEntries` callers and for "three PATTERN" wording, to find stale counts and other readers.
