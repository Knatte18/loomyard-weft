## Changes

- **F1** (fixed as proposed), `internal/fabricengine/worktreeseam_integration_test.go`.
  `TestWorktreeSeam_Events` now takes the copies behind its later subtests before `InstallWorktreeSeam`:
  `boltOnlyOrigin`, `failingPullOrigin`, `parentPairOrigin` and `reconcilePair`.
  The subtests read these instead of calling `newOriginHub` and `newMovesPair` after the install.
  `boardkit.InstallReplay` moved to the top of the test, before the copies, so the `movesTemplate` build runs under replay as it did before.
  The first-subtest copy at the top stays where it was, before the install.
  Commit `ffac7c18`.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-proposal: no findings.
- fix-consistency, MEDIUM: `buildReconcileFixture` copies `hubforge.Shape{Anchor: "."}` after the install in two subtests, so that template's first build could run under the fake seam.
  Fixed as proposed: one `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})` before the install builds the template first.
  The tier-1 gate on `./internal/fabricengine` passes.
  This is a second commit.

## Disputed

None.

## Deferred

None.
