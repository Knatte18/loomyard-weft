# Plan round 2 — fixer report

All eleven findings were validated against the code and the decision record, and all hold; each is fixed.
`lyx loom validate-plan` passes on the result (informational findings only).

## Fixed

- **F1 (BLOCKING), card 12:** the standalone keep-reset moved out of `internal/websterengine`.
  `resetCmd`'s standalone branch calls `gitrepo.New(c.geom.WorktreeRoot).ResetKeep(plan.SHA)` itself, as its pair branch calls `fab.ResetPairCode`, so `cmd/lyx/rawgitmutation_test.go`'s scan of websterengine stays clean.
  The "file header names the exception" sentence is gone, and `reset.go`'s header stays true.
  The per-path way forward is built by a pure `KeepResetRefusal` in `reset.go`, fed the overwritten paths `resetCmd` reads through `gitrepo` (`WorktreeChangedFiles` ∩ `ChangedFilesSince`).
- **F2 (MEDIUM), cards 11, 12, 14:** each card now lists its `refusal-spec.md` rows by name, with a delete or reword per row.
  Card 11: `reset: unknown --to`, `reset: run busy`, and `reset: target not an ancestor of HEAD` (the `pre-fix` clause extended to the new targets), plus the new rows.
  Card 12: deletes `reset: no recorded target, start` and `reset: standalone mode`; rewords `reset: target commit missing`, `reset: target not an ancestor of HEAD`, `reset: dirty tracked path the run did not write` and `fresh: recorded starts diverge`'s resolving clause; adds the standalone keep-refusal row and the pending-findings guard row.
  Card 14: lists every row carrying the reset-then-fresh pair, including the three audit rows' "toward the `--fresh` route" phrasing, names the rows it leaves unchanged, and states its order against cards 12, 13, 16 and 19.
- **F3 (MEDIUM), cards 12, 14:** the factored archive keeps `freshPendingDrop`'s guard.
  Every reset-to-start path runs those checks against the tree after the move and archives only when they pass, so no reset drops a finding `run --fresh` would refuse to drop.
  For the reset caller, each way forward names a step other than the reset itself (rm, restore-plan or rebaseline, `git checkout <start> -- <path>`, accept-audit), then re-running the reset.
  The bound is stated.
  Card 14 rewords only the `run --fresh` caller's texts.
- **F4 (MEDIUM), card 20:** `CommitPlan` commits a new `planCommitPathspec`, which mirrors `reworkCommitPathspec`.
  It adds webster's directory when that directory, or an `archive-*/webster` directory, holds a file, so the moved-from deletions land in the same commit.
  `TestPlanCommitPathspec` is a table test of the three cases.
- **F5 (MEDIUM), cards 11, 14, 19:** the missed sites are added.
  Card 14 lists `webstercli/cli_test.go` (the two `--fresh` rows of `TestRebaselineCmd_Refusals`).
  Card 19 lists `persistPlanFingerprintRebaseline`, which now copies the frame hash, and `cli_test.go` (`TestPersistPlanFingerprintRebaseline`, plus the done-batch row it rewrites again).
  Card 11 lists `Command`'s group help `Verbs:` line.
- **F6 (LOW), card 12:** one assertion per behavior.
  The archive is asserted in `TestResetCmd`'s existing start step, and the keep semantics stay only in card 10's `TestResetKeep`.
  The standalone surface asserts only the per-path way forward and `uncommitted`.
  `reset_integration_test.go` is named as driving `PlanReset`, for the octopus no-common-ancestor cause only.
- **F7 (LOW), card 11:** the refusals that read only state, report files, the reed seam or a fake `Git` move to the untagged `reset_test.go` over `fakeGit`.
  `TestPlanReset` keeps the resolved SHAs and the ancestry and dirty-path guards over a real repository.
- **F8 (LOW), card 20:** the collision row is dropped, and the card states why `firstFreePlanArchivePath` makes it unreachable from this caller.
  The card also states the retry behavior accurately: a retry archives into the next free directory.
- **F9 (LOW), card 8:** the derivation point is stated: the stencil as it stands when card 8 runs.
  Drift from cards 13 and 17's rungs is left to the `orientation` refit, per the auto-mode assumption.
- **F10 (LOW), card 2:** `Breakdown.Base` is dropped and `Breakdown` leaves the target list; `Breakdown.Startup` includes the base through `breakdownOf`.
- **F11 (LOW), card 6:** the builder returns a `configLoadRefusal` type, and the pre-run decides the flag by `errors.As` on that type, never by `errors.Is(ErrInvalid)`.
  A test step asserts a malformed `shuttle.yaml` carries no `config_invalid`.

Gate fallout: new symbols that no other card references are named in Intent prose with their files under `Edit:` instead of as `plan:` handles, which the `handle-unreferenced` check rejects.
This applies to `KeepResetRefusal`, `configLoadRefusal`, `planCommitPathspec` and `ArchivedPlanWebsterDirs`.
Card 6 now targets `internal/webstercli/wiring.go` in place of its three member targets there, and card 20 targets `internal/loomshed/planwrite.go` and `internal/loomcli/wiring.go`.

## Disputed

None.

## Deferred

None.
