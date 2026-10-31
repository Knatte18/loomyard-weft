```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
1. A Worktree-Teardown resume with the task worktree absent runs no reconcile and resumes when the local board shows the run owned here. `TestOwnerPreflight_TeardownWithPairGone/OwnedHereResumes` (`owner_test.go:181`) pins it.
2. The same resume with the board showing another owner refuses naming that owner and `lym adopt some-beam`, without "once origin answers". `.../OwnedElsewhereRefusesNamingTheAdopt` (`owner_test.go:186`, assertions at 217-228) pins it.
3. The skipped reconcile leaves no origin-owed marker. The `OriginOwed` assertion at `owner_test.go:233` pins it.
4. A resume at any other producer still runs the reconcile. `.../OtherRowRunsTheReconcile` (`owner_test.go:192`, producer `NameLand`) pins it.
5. The unreachable-origin refusal keeps its "once origin answers" hint after the `refuseResumeUnreachable` to `refuseOwnedElsewhereOnLocalBoard(retry)` refactor. `run_test.go` row `unreachable_origin_with_the_local_board_owned_elsewhere_refuses` pins it.
6. The failed-deletion error names `lym batten run <slug>`. `wire_test.go:840` (`TestFinishRemoval`) pins the wording, and `lifecycle_integration_test.go:705` pins it through the real teardown.
7. A Worktree-Teardown resume with the task worktree present still runs the reconcile (acceptance criterion D1, second bullet): none.
8. A failure to read the worktree's presence, where `taskWorktreePresent` returns an error that `ownerPreflight` returns unchanged (`owner.go:236-239`): none.
9. A Worktree-Teardown resume with the pair gone and an unowned entry resumes (the "or unowned" half of the first D1 criterion): no row in the new table. The refusal function's unowned handling is pinned only through other call paths.
10. A Worktree-Teardown resume with the pair gone on a receiver with no owner seams returns nil without a reconcile (`owner.go:242` via `owner.go:279`): none.

**Findings**

- **MEDIUM, design** — `internal/battencli/owner_test.go:170-239`. Nothing pins that a Teardown resume with the task worktree present still reconciles, although the acceptance criteria require it. `OtherRowRunsTheReconcile` uses `NameLand` with the worktree absent. If the skip dropped its `!present` check and skipped for every Teardown, no test would fail. Evidence: `owner.go:235-243`, and `newFakeReceiver` (`run_test.go:41-48`) builds no task worktree directory. Fix: add a row with producer `NameWorktreeTeardown` and the task worktree directory created, expecting a reconcile.
- **LOW, design** — `internal/battencli/owner_test.go:173-197`. The "owned here or unowned" criterion has only an owned-here row, so the unowned and run-held outcome is not pinned on the skip path. Fix: add a Teardown row with `ownedBy("here", "", true)` expecting a resume with no reconcile.
- **LOW, design** — `internal/battencli/owner.go:236-239`. The error path when `taskWorktreePresent` fails during the Teardown check has no row. It would need an unreadable path or a bad anchor. Fix: add a case, or accept the gap because the error is a plain passthrough.

**Tool calls:** none made. The manifest already held the diff, `owner.go`, `owner_test.go`, `newFakeReceiver` and `taskWorktreePresent`.
