# Round 2 fixer-report

## Changes

All changes are in `internal/gatecli/testmap_integration_test.go`.

- F1, fixed as proposed (commit "fix F1"):
  the new subtest `ASkipAfterTheTestsCountsAgainstTheCap` runs `dirtier`, `follower` with a cap of 1 and a file dirtied under the tests.
  It asserts `Unbuilt` is 1, `dirtier` skipped after its tests and `follower` never tried.
  The reviewer's alternative was taken as a separate subtest, because the existing subtest also needs `follower` walked under cap 10 to pin the fresh status read.
- F2, fixed as proposed (commit "fix F2"):
  the stand-in gained `testMapLandFileEnv` (a file of the linked package to create or change) and `testMapRemoveFileEnv` (a file to remove), handled by the new `landInLinked`.
  The new subtest `ALinkedFileAddedOrRemovedUnderTheTestsWritesNoMap` has an `Added` row and a `Removed` row, each asserting the skip cause `files changed under its tests: linked/landed.txt` and no map written.
  `appendAndCommit` now shares the new `gitSteps` helper.
  The own-package added and removed branches stay untested: the review's fix named the linked variant, and both branches are one code path for every package.
- Self-check fix (commit "fix self-check finding"): the `landInLinked` comment broke under 120 characters.
- Gate fix: the `landInLinked` comment wrapped at a fixed column, so it is rewritten as two sentences, one per line.

Verification: `lym gate test --tier 1 ./internal/gatecli -- -run TestGateTestMap` passed after each change.

## Self-check

- fix-code-correctness, LOW, the `landInLinked` comment line passes 120 characters: fixed as proposed.
- fix-consistency, LOW, the same comment line: duplicate of the finding above, fixed with it.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
