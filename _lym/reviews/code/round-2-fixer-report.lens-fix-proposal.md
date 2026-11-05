```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1** (notice wording). The payload `{"slug":"<dependent>","depends_on":[...]}` matches the `set-deps` contract (`internal/boardcli/cli.go:1034-1037`): `depends_on` is required and `slug` is the selector. `TestSketchFinish_TellsTheParentOnlyOfARefusedSketch` (`commitstatus_integration_test.go:167`) still matches on the substring `lym board set-deps`. The added text has no `%`, so the `Sprintf` arguments are unaffected.
- **F2** (package doc order). The new wording matches the order the review gave: the start guard judges first, then `claimStart` removes the run after its owner check. I found no sibling text that contradicts it in what my context shows.
- **F3** (test row). The row has a raddle entry with its own recipe, so `startClearsMotifRun` never opens the board. `ReadChildRecipe` hits the directory at `ChildRecipeFile` and returns a non-`ErrNotExist` error, so the `unreadableDoneRunStart.Fail` branch runs and `clears` stays false. `wantRefused: true` therefore holds. The unreadable record never lands in a git index or a cleanup path, and a directory fails to read even as root.

One limit on F3: the row pins that an unreadable judgment leaves the start refused. It cannot catch a regression that would clear on error, because `startClearsMotifRun` returns `false` with the error. That is in scope as the review asked it.

Tool calls (2 of 6 allowed):
- A single Bash call that ran `grep` on `internal/boardcli/cli.go` and `internal/loomcli/*_test.go`. My context lacked the `set-deps` payload keys and the notice test's assertions.
