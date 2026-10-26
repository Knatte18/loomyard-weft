```yaml
lens: code-pattern-fit
outcome: reported
```

Task: check the diff against the repository's rules (PATTERN.md entries, CLAUDE.md), as the code-pattern-fit lens.

**Rules checked**
- `PATTERN-leaf-packages`: held. `patternchanges.go` adds only `regexp`/`strings` use. The `internal/pattern` import is in the `_test.go` file only, as before.
- `PATTERN-sole-parsers`: held. `planparser` calls `pattern.EntryRuleText` on lines `CardPatternEntries` already extracted. `discussionparser` still parses its own bullets.
- `PATTERN-test-speed`: held. Rows were added to `TestCheck`, `TestValidate_PatternEntryLineCap` and `TestValidate_PatternChangeOverCap`, and the pin test was extended. The one new function, `TestEntryRuleText`, covers the new exported `EntryRuleText`. No tier tags changed.
- `PATTERN-refusal-way-forward`: held. The reworded discussion and plan-gate messages still name the way forward ("shorten the text or move its detail to the entry's background file").
- CLAUDE.md "Docs land in the same commit": mostly held, with one miss, F1 below. The `pattern`/`planparser` `doc.go`, both specs and the plan stencil were updated.
- CLAUDE.md "Terminology" and "Markdown: semantic line breaks": held. The spec and stencil edits keep one sentence per line.
- Declared PATTERN changes: nothing to land. The diff touches neither `PATTERN.md` nor `pattern/`, and the decision record has no `## PATTERN changes` section.
- Commit messages and departures: held. Neither commit has a `Pattern-Departure:` trailer, and none is needed. The second commit's subject is plain.
- Comment conventions: held. New doc comments open with the symbol name, are self-contained, and have no end-of-line comments. Wrapping is the lint gate's concern.
- Scope against the board entry and decision record: held. The overview byte cap is untouched, the check names are kept, and the cap value is 280.

**Findings**

F1: LOW, `consistency`, `internal/discussionparser/doc.go:11`
- Summary: the package doc still says the check refuses a bullet "whose PATTERN.md entry line would run past the PATTERN format checker's line cap".
- Problem: the cap is now a rule-text cap. This is the doc the decision record scopes under "the package docs", and CLAUDE.md requires it to change in the same commit.
- Evidence: `internal/discussionparser/doc.go:11`, which my earlier grep for "line cap" surfaced. The diff does not touch this file.
- Fix: reword it to "whose rule text, the entry text without a trailing `(test)` or background link, would run past the PATTERN format checker's rule-text cap".

No other rule broken.

**Tool calls:** none. The diff and the grep results in my context were enough.
