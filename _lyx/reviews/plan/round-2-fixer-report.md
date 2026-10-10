# Plan-Review round 2 — fixer report

Every finding was checked against the code and the decision record before fixing.
`lyx loom validate-plan` passes over the fixed plan, with only informational `create-new-unit` rows.

## Fixed

- **F1** (BLOCKING): confirmed.
  `runReconcile` builds one `rec` and emits it under `mutations`, and pushes through `Bolt.Push` at `internal/fabriccli/fabric.go:774` (hub-wide config) and `:842` (warp binding).
  Card 13 now creates `Bolt.PushRecorded(opts SyncOptions, rec *Mutations) error`, which holds the drop-and-retry and passes the caller's `rec` to `dropSeedCommits`.
  `Bolt.Push` keeps its signature and calls it with a local `Mutations`, logging the dropped SHAs at Info for callers without an envelope.
  Card 13 also edits `internal/fabriccli#runReconcile` to switch both pushes to `PushRecorded` with the handler's `rec`, and updates the warp-binding push comment that says the push records nothing.
  The switch sits on card 13, not card 11 as the review proposed, because card 11 listing a symbol card 13 creates would break card order (`uses-later-target`).
  A second card editing `runReconcile` after card 11 is legitimate sequencing per the plan spec.
  The test becomes `TestBolt_PushRecorded_DropsSeedCommitsOnRebaseConflict` and asserts the passed `rec` holds one `KindCommitsDropped` entry on the drop and nothing on the conflict.
  `doc.go` names which push records the drop and which only logs it; the overview's Card Index line 13 is updated.
- **F2** (MEDIUM): confirmed: `internal/fabricengine/export_test.go` is the package's re-export shim for the gate's unexported executors.
  Card 7 lists it under `Edit:`, and the Intent states it re-exports `dropSeedCommits` and `boardDropRequest` for the foreign-board test.
- **F3** (LOW): card 7 now states the fixture: a linked worktree of the same `hubforge` hub's records repo, added with `gitkit.MustRun` and `git worktree add` at `<t.TempDir()>/elsewhere/_board`.
  That is the existing precedent in `add_runrecords_integration_test.go` and `add_rollback_adopt_test.go`.
- **F4** (LOW): confirmed: `BinaryLabel` reads `buildvcs.Running()`, not the identity passed to `seedStencilsAt`.
  Card 8's scenario step now asserts each subject equals `SeedCommitMessage(<subtree>, BinaryLabel())` computed in the test process, and says the fixed identity drives only the writer keys.
- **F5** (LOW): confirmed locally on git 2.53: a plain `git cherry-pick` of a commit whose change the branch already carries exits 1 ("The previous cherry-pick is now empty"), and `git cherry-pick --empty=drop` drops it with exit 0.
  Card 5's `CherryPick` now runs `git cherry-pick --empty=drop <sha>`, so the replay matches `pull --rebase`, and `TestCherryPick` gains a row asserting such a pick succeeds with HEAD unmoved.
  Card 7's replay sentence states the rule.

## Disputed

None.

## Deferred

None.
