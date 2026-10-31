```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for F1, F2 and F3 and check the fix covers every member.

**Findings**

1. **LOW**: `internal/dobbyengine/doc.go:97-99` is stale after the F1 change in `internal/dobbyengine/starts.go:89`.
   - The doc bullet says dobby records the refusal and posts one notice for "a released task whose slug has a finished run record".
   - It now does so only when the task is also not owned elsewhere, has no pending dependencies, and is not covered by a pause.
   - The `startReady` doc comment (`starts.go:55-59`) and the `heldStart` doc comment (`starts.go:212`) also state the hold without the new condition.
   - The fix's edits (`git diff HEAD~3 --stat`) touched only `starts.go`, `view_test.go` and `beamcli/cli.go`, so these three descriptions were left as they were.

**Sets checked with no gap**
- **F2:** a repo-wide grep for the waits wording ("waits on", "ready to start", "refused start") found one other site beyond the review's `battencli/cli.go:27` and `beamcli/cli.go:73`. That site is `battencli/cli.go:27`, which the review calls already updated. No other help text or doc enumerates the waits.
- **F3:** the only `StartRefusal` rows in the view tests are the `finished` row and the new `ATaskWithAStartRefusalTakesNoMaxRunsSlot` row.
- **F1:** `starts_test.go` has no row where a refusal-carrying task is also owned elsewhere, has pending dependencies or is paused. The review did not ask for one (F1 is a design finding), so I raise no finding.

**Tool calls**
1. A grep for the waits wording across `*.go` and `*.md`, to find other sites that list what a task waits on; my context held only the three files the review named.
2. `git diff HEAD~3 --stat` plus a grep of `starts_test.go` for `StartRefusal` and `OwnedElsewhere`, to confirm which files the fix touched and which test rows exist; my context did not show the diff or the whole of `starts_test.go`.
