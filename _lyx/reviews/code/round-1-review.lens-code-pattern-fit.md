```yaml
lens: code-pattern-fit
outcome: reported
```

Task: lens `code-pattern-fit` — check each changed file against the repo's rules (PATTERN.md, CLAUDE.md, rubric).

**Rules checked**

- `PATTERN-sole-parsers`: held. `checklistSkippedRows` reads `status.History` entries already parsed by `shedengine.Status`. It parses nothing new (`internal/loomcli/checklist.go`, end of file).
- `PATTERN-tolerant-records`: held. The diff adds no JSON read or write and no refusal of unknown keys.
- `PATTERN-test-speed`: held. The new rows sit in the existing table-driven `TestRenderChecklist`. They use injected clocks (`minuteMark`, `nowMin`). The file keeps its `//go:build tier0` tag (`internal/loomcli/checklist_test.go:1`). The new rows cover the skipped row, the skipped-and-visited row and the skipped-only review segment, and each pins behavior no earlier row covered. The existing skipped row was reworded in place, not duplicated.
- `PATTERN-test-isolation`: held. The new rows use no git or tmux.
- CLAUDE.md "Docs land in the same commit": held. `internal/loomcli/doc.go` names the skipped box in the checklist paragraph. No module, CLI, execution-stack or invariant change is made, so `docs/overview.md` and `PATTERN.md` need no edit.
- CLAUDE.md "Markdown: semantic line breaks": held. The only `.md` content in the range is `_lyx/` records, which are not the subject. The `doc.go` paragraph keeps its single long line, as before.
- Comment conventions (`scribe:golang-comments`, rubric "Comment-convention compliance"): held. The new doc comments on `checklistSkippedRows`, `checklistBox` and `skippedEntry` start with the symbol name. They are self-contained and have no end-of-line comments. The inline comment in `checklistReviewBlock` explains a non-obvious why. Line wrapping is not a finding under the rubric.
- CLAUDE.md "Notes go in git" and "Worktrees": held. The diff touches only `internal/loomcli/*`, and nothing writes outside the worktree.
- Declared PATTERN changes: none declared, and the diff has no `PATTERN.md` or `pattern/` changes. Nothing to land.
- Commit messages and departures: held. There is one commit and no `Pattern-Departure:` trailer, and none is needed. The commit trailer matches the attribution convention. The commit body gives the reason for the change.
- Unrecorded parent correction and own-entry board edits: not applicable. The diff and the change description show neither. The `[=]` move is a recorded decision in the record.
- Stencil wording: not applicable. No stencil changed.

**Findings:** none for this lens.

**Out-of-lens notes**
- `checklistVisitsOf` is documented as treating a skip as no visit. The affected-set and correctness lenses should confirm that no other reader of the halted glyph `[-]` exists. My own check shows the only remaining `[-]` strings are the new skipped boxes.

**Tool calls (2)**
1. `head -5 internal/loomcli/checklist_test.go`, `git diff --stat -- PATTERN.md pattern` and a grep for `Pattern-Departure` in the range's log. My context lacked the test file's build tag, the pattern-diff stat and the trailer check.
