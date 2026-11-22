Lens fix-code-correctness: I ran each changed branch of the fix step by step with concrete inputs, and I found one weak test row.

**MEDIUM — the `RunShedRunningAfterRunShedPausedPushes` row does not cover the marker-producer conjunct.**
- Changed text: the F3 row `{name: "RunShedRunningAfterRunShedPausedPushes", calls: {Run-Shed paused}, {Run-Shed running}}` in `internal/battencli/commitstatus_test.go`.
- It was meant to pin `slices.Contains(preRunRows, marker.Producer)` at `commitstatus.go:264`, and it passes with that conjunct removed.
- Evidence: after `{Run-Shed paused}` the marker state is `paused`, so the `marker.State == running` conjunct already routes the second call to `core`, with or without the producer check.
- A row that does pin the conjunct needs a running Run-Shed marker followed by a different start row, such as `{Run-Shed running}, {Seed-Child running}`.
  - Without the conjunct, that sequence takes `startCore` and pushes once.
  - With it, the sequence takes `core` and pushes twice, so it expects 2 commits and 2 pushes.
- The `marker.State == running` conjunct is also unpinned by this row, because the producer check masks it. `HaltedStartRowPushes` ends on `st == failed`, which does not reach the marker-state conjunct either.

Everything else holds:
- **`seedFreshStatus`:** `seeded` is true only for a successful write of an absent status. A `preDriveRefusal` from the closure still reaches `errors.As` unwrapped in `battenPreStepUnmarked`. A found status makes no commit call, and a nil `c.commitStatus` makes none either.
- **Wiring:** `c.commitStatus` is set before `shedPaths`, and `CommitStatus: c.commitStatus` is the same seam instance.
- **Marker and push-pending logic:**
  - The merge-skip path leaves the marker unchanged, because `commitMade` is reset to false before `core` and `afterCommit` does not run.
  - A failed push leaves the marker pending.
  - An owed push that lands, or a clean push, rewrites it as not pending.
  - A marker without `push_pending` decodes as false.
- **`TestNewCommitStatusSeam_RetriesAFailedStartChainPush`:** it fails if the `!marker.PushPending` check is removed. Call 2 would take `startCore` and the push count would be 1, not 2.
- **`TestFreshStartCommitsWorktreeCreateThroughTheSeam`:** it fails if the `seedFreshStatus` commit call is removed, because commits would be 0.
- **New rows for the `startCore` marker write and the Worktree-Create exclusion:** both fail with their defect in place.
  - Without the marker write, the third call commits again, giving 3 commits.
  - Without the exclusion, the second call takes `startCore`, giving 1 push.
- **Admit assertion:** `admits == wantPushes` holds for every existing row, and it fails if `startCore` took admission.

Tool calls: none, since my context already held every changed file.
