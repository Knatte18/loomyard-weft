```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F6 change set through the code-correctness lens, using only the context I already hold.

**Findings**

- **MEDIUM** — `TestReconcileOptionsWaitOnAPushLock` (`internal/adoptcli/deps_test.go`) and `reconcileOptions` (`internal/adoptcli/deps.go`): F2's test passes with the defect in place.
  - The defect is the seam dropping `PushLockWait`. It would come back by reverting `Reconcile` to `fabricengine.ReconcileOptions{}`, or to a literal without the field.
  - The test calls only `reconcileOptions()`, which stays correct and unused in that case. Nothing ties `hubDepsAs`'s `Reconcile` closure to the helper.
  - The review's own proposal was to drive the seam over a hub with a held push lock, or to expose the options the seam actually passes.
  - A weaker pin is unavoidable unless the seam takes its options from a field the test can read.

- **MEDIUM** — the `CodeBranchFile` row in `TestDropLocal` (`internal/fabricengine/droplocal_integration_test.go`): it passes with the defect in place.
  - The defect is the records-side guard `target.side == failedPushSideRecords` being removed, so `untrackedOnPaths` runs on the code side too.
  - The row's `scratch.txt` is not in the code side's `touched` paths. `touched` holds only `local-code.txt`, which is committed. `untrackedOnPaths` would therefore return nothing for the code side, and the drop would succeed with the file kept.
  - To exercise the guard, origin's code tip must also track `scratch.txt`. The drop would then be refused by git rather than succeed with the file kept, so the row's assertions would need changing as well.

- **LOW** — the `IgnoredFileOnAPathOriginTracks` row, same test: its name and subtest title say the ignored file "is not removed", but the assertion is that origin's content overwrote it (`want = "origin\n"`).
  - The row does catch dropping `--exclude-standard`, because `KindPathRemoved` would then be recorded.
  - It does not show an operator's ignored file surviving, which is what F1 asked for. Git overwrites the file silently.
  - The wording should say "is overwritten by git, not removed by the drop".

**Checked and sound, would fail with the defect in place**

- `TestOwnedUntrackedTrackedAt_RefusesATargetTheIndexTracksOrTheCommitDoesNot`: all three rows hit distinct branches of `pathOwnershipUntrackedTrackedAt`.
- The `WeftFileOriginDoesNotTrack` row of `TestDropLocal`: if the `slices.Contains` filter were dropped, `scratch.md` would be listed and refused as "not tracked at", so `DropLocal` would error.
- The new write-lock subtest in `TestReconcileOrigin_Moves`: any wait on a write lock would make `sleeps` nonzero.
- `HeldBackRecordWhoseStopStillHoldsStopsAgainWithTheSameWayForward` and `HeldBackRecordRejudgedKeepsItsPausedSlugs`: the second row's `w.record.Live` check catches a cleared `Inputs.Paused`, because the pause step would then pause alpha again.
- No caller of a changed signature is affected. `reconcileOptions` is new and private, and the `doc.go` and `overview.md` edits touch only text.

**Tool calls made:** none. My context already held every changed file.
