```yaml
lens: fix-consistency
outcome: reported
```

Task: check the consistency of the fixer's two new test rows with their surroundings.

No findings.

- The new rows sit beside `ArchivedJudgementIsNoChange` and use the same row shape, naming and fixture paths.
- The expected list is in sorted order. `old/` sorts before `other-`, because `/` (0x2F) is below `t` and `judgement-` sorts before both.
- The code-checkout row uses `f.code.Untracked` and `f.code.Root`, matching the code side of `Changes`.
- The `export_test.go` wiring row and the doc comments are untouched, and the change adds no twin to update.

Tool calls (1):
- `git diff HEAD~1 -- internal/driftengine/judgesnapshot_test.go`: to confirm the committed text after the formatter ran, which my context did not show.
