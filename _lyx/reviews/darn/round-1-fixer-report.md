# Fixer-report, round 1

## Changes

- F1 (`internal/configcli/hubwide.go`): `reconcileDefaultLayer` snapshots the board dir's config files after the dry run and, when the apply call fails, restores the files that differ from the snapshot (`snapshotBoardConfig`, `restoreBoardConfig`).
  The doc comment now says a parse or migrate failure leaves every file unwritten and a part-way failure is undone; `internal/configcli/doc.go` carries the same wording.
  `TestRestoreBoardConfig` pins the restore of a changed, a removed and a created file.
- F2 (`hubwide.go`): the commit error and the push error are tracked apart; a push failure alone reports the files as committed in `_board` but not pushed.
- F3 (`configcli.go`): `runReconcile` runs the board-dir refusal before the layer check.
- F4 (`configcli_integration_test.go`): a subtest makes the apply fail on a read-only board config dir and asserts exit 1, a `reconcile:` error, an unchanged `loom.yaml` and an unchanged `_board` HEAD.
  It skips on Windows and as root.
- F5 (`configcli.go`): the config help names `--layer default --apply` from the prime for the board dir's files and `reconcile --apply` for a worktree file.
- F6 (`internal/configengine/config.go`): the `missingBaseError` comment and message name `lyx config reconcile --layer default --apply` from the prime as a writer of a per-worktree repo default.
- F7 (`config.go`): `warnUnknownKeys` returns early for a spec with `WholeEntries`.
  The other `Prune` caller, in `internal/configsync/configsync.go`, runs only for modules that are not `SeedOnly`, and the one `WholeEntries` spec is seed-only, so it needs no change.
- F8 (`configcli_integration_test.go`): a dry run from the task pair expects exit 0 and an unchanged board `loom.yaml`; a refusal row covers `--layer local` from the board dir (F3).

## Gate repair

The verify gate failed `TestCLIForm_EveryCommand`: the F5 help text pushed the `lyx config` Long page to 2133 bytes, over the 2000-byte cap.
Fixed by dropping the per-round model-key example from the help page, which `internal/configcli/doc.go` already carries; `./cmd/lyx` and `./internal/configcli` pass.

## Fixed differently

- F4: the review proposed making the apply or the commit fail and asserting the F1 and F2 behaviour.
  The integration test covers the apply failure only; the part-way restore is pinned by a unit test, because a read-only dir stops every write and cannot leave a partial one.
  The commit and push failure branches have no test; no commit seam exists to fake them.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness, MEDIUM: `restoreBoardConfig` rewrote every snapshotted file, so a failure before any write reported false "may be invalid" restore errors. Fixed: it restores only files that differ.
- fix-code-correctness, MEDIUM: the failed-apply integration test passes without the restore. Fixed differently: `TestRestoreBoardConfig` pins the restore; the integration test stays as the error and no-change check.
- fix-proposal, LOW: same defect as the first fix-code-correctness finding. Merged, fixed.
- fix-consistency, LOW: `doc.go` kept the old "fails to reconcile" wording. Fixed.
- fix-consistency, LOW: the `missingBaseError` hub-wide route does not name `--layer default`. Not changed: the review's fix names the per-worktree repo default only, and the comment remains true.
- fix-consistency, LOW: the `load` doc comment says `lyx config reconcile` is the only writer. Not changed: it was already untrue before this fix, and the fix did not make it worse.
- fix-affected-set, LOW: `configsync.go` `Prune` caller omits `WholeEntries`. Not changed: it is guarded by `!m.SeedOnly`, and the only `WholeEntries` spec is seed-only.
