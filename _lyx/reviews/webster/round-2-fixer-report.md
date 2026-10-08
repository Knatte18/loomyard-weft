# Webster round 2 — fixer report

Every finding was validated against the code and fixed; each fix is its own commit.
The plan's verify set (`go build`, `go vet` under the integration, tmux and llm tags, `go test` untagged and `integration`) passes, and `lyx loom lint-comments` reports ok.

## Fixed

### F1 — snapshotConfig treats an unreadable file as absent (`b34a0b3fa`)

Confirmed: the registry loop mapped every `os.ReadFile` error to `nil`, and `restoreConfig` answers `nil` with `os.Remove`.
`snapshotConfig` now returns an error: only `fs.ErrNotExist` maps to an absent file, and any other list or read failure, in the dir loop or the registry loop, is returned naming the path.
`reconcileWorktree` fails the worktree with a `*WorktreeError` before `ReconcileAll` writes anything, and `reconcileHubWide` returns the error from its `CommitWritten` write func, before `ReconcileHubWideAt` runs.
`doc.go` states the new failure.

Test: `TestEnsure_UnparseablePairConfigNamesPairAndFile` became the table `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas`.
Its new row places a directory at the pair's `loom.yaml`.
The row failed before the fix, because the restore removed the directory.
Every row asserts that the error names the worktree and the file, that the file still exists, that the repo's status is unchanged, that no stamp is written, and that a walk after the repair stamps and leaves the tree clean.

### F2 — a restore after a landed commit dirties the records worktree (`f1cb401ef`)

Confirmed: `commitBothSides` returns `WeftCommitted` with a `*PartialCommitError` when `RecordCorrespondence` fails after the weft commit lands, and `reconcileWorktree` discarded the result and restored anyway.
`reconcileWorktree` now keeps the `CommitResult`, and when `res.Committed()` is true it returns the `*WorktreeError` without restoring.
The function comment and `doc.go` state the exception.
No test was added, since this is not a coverage gap (`PATTERN-test-economy`), and no existing test asserted the old behavior.

### F3 — reconcileHubWide's restore is untested (`64df647cf`)

This is a coverage gap.
I added the `unparseable hub-wide config` row to the same table.
The fixture's `board.yaml` is already current, so the row first commits a stale `board.yaml` on the board, which the walk writes before the unparseable `fabric.yaml` fails.
I ran the row once with the restore disabled, and it failed with ` M _lyx/config/board.yaml`, so it does reach the restore.

## Disputed

None.

## Deferred

None.
