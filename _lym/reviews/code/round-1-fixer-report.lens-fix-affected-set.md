```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1 (`runDir` existence check), F2 (prompt-failure row) and F3 (Info log reword) cover every member of the sets they touch.

No findings.

- **F1 callers.** The only production caller of `RunButton` is `internal/reedcli/button.go:77`, which passes `buttonRunDirectory`. The only other `anchorOf` stub is the integration test at `internal/reedengine/buttonclick_integration_test.go:59`. It returns `dir` unchanged, and `dir` already passes the `spec.Dir` stat, so the new `runDir` check cannot reject it.
- **F1 test rows.** Every `TestRunButtonVia` row that sets `anchorMoved` now creates `runDir` with `MkdirAll`. The one new `anchorGone` row leaves it absent. The `exeErr` row now has `anchorMoved: true`, so the directory exists and it reaches the prompt-failure branch.
- **F2.** The prompt-failure message is the only text in `buttonclick.go` that names `runDir/.lym/logs`, and the reworked `exeErr` row now pins it to the moved anchor.
- **F3.** A grep for the old Info text, "is not its worktree's anchor", finds no other occurrence in Go or markdown files outside `.lym/` and `_lym/`. There is nothing asserting on it, so the reword leaves no stale twin.

Tool calls:
1. `grep -rnE "anchorOf|is not its worktree's anchor|RunButton\("`: lists every user of `anchorOf` and `RunButton`, and every site of the old log text. My context held only the changed files.
2. `grep -rn "buttonRunDirectory"`: finds the production anchor reader behind the one `RunButton` caller, whose behaviour my context lacked.
