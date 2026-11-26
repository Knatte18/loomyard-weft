```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the searches behind the set-premise findings (F5, F7, F8, F9) and check that the fix covers every member.

Unchecked list: none of F5, F7, F8 or F9 is left unchecked. F1–F4, F6 and F10–F13 do not rest on a set.

**LOW**
- **Changed text:** card 1, the `internal/boardcli#storeVerbs` line under Edit and the Intent sentence on the `run_settings` help line.
- **Summary:** card 1 changes the help line, but `internal/boardcli/help_test.go`, which pins that help's key list, is left out. No test pins the new `beam.run_watch` entry, and the help test's existing rows go stale.
- **Evidence:** `internal/boardcli/help_test.go:62` asserts `"beam.land_notice"` in the board help. That is the help line card 1 now extends with beam.run_watch, and card 1 lists no `internal/boardcli` test under Edit.

Sets checked with no gap:
- **F5:** no `_test.go` calls `CheckLocalValues`, so `TestPrepareRunSettings` is the only test that reaches it, and card 1 now edits it.
- **F7:** the done-run-clear wording appears at `internal/dobbyengine/doc.go:191` and in the `clearDoneRuns` doc comment (`donerun.go:15-19`). Card 3 now covers both.
- **F9:** only `docs/overview.md` and `contracts/orchrole/orch-role.md` name `[dobby notice]`. Cards 4 and 7 now edit both.
- **F8:** the other sites that name the per-task keys are left alone correctly. `pattern/PATTERN-config-layers.md:125` is already in card 1. `contracts/orchrole/role-beams.md:56`, `internal/battencli/overviewdobby.go:36`, `internal/battencli/cli.go:62` and `internal/boardengine/store.go:263` each name a single key, not the full per-task list.

Tool calls:
1. Grepped for `beam.land_notice` across Go and markdown, to rerun F8's search for sites that list the per-task keys. My context held only `cli.go`.
2. Grepped `docs`, `contracts` and the package docs for `[dobby notice]`, and the tests for `CheckLocalValues`, to rerun F9's and F5's searches.
3. Grepped `doc.go` and `donerun.go` for the done-run-clear wording, to rerun F7's search.
