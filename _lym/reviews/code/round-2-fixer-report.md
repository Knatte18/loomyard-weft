# Round 2 fixer-report

## Changes

- **F1 (MEDIUM), fixed as proposed.** Rewrote `_lym/landing/summary.md`: the `--subpath` check runs before the reset through `checkSubpathBeforeReset`, shares `fetchWeftBranchTip` with the probe, and the Tests section lists the new tests.
  Not committed: the round's rules bar git commands against `_lym` paths, so the file is changed in the working tree only.
- **F2 (MEDIUM), fixed as proposed, as one new table test.** Added `TestCloneHub_ResetSubpathCheckReadsRecordedAnchor` in `internal/fabricengine/warpbinding_clone_integration_test.go` with three rows: marker only on `main-weft`, `main-weft` without a marker plus a marker on the default branch, and a marker on a default branch that is not the warp branch (not adopted, so the reset with a warp-present `--subpath` succeeds).
  Rows one and two assert the refusal names the recorded anchor and the sentinel survives.
  Committed `3ef573e9`.
- **F3 (LOW), fixed with the review's alternative.** Added `t.Parallel` to `SecondCloneDerivesShortname` and `DisagreeingShortnameRefuses`, which use only `weftURL` and a fresh `t.TempDir()`.
  The comment above `ResetWithDisagreeingShortnameKeepsHub` stays and names it and `ResetKeepsRecord` as the serial pair sharing the hub.
  Committed `20f254c1`, then reworked in the self-check commit below.
- **F4 (NIT), fixed as proposed.** `livestate_doc_test.go` row (8) names `checkResetTarget` instead of `resetHub`'s pre-flight.
  Committed `52efca26`.

The tier-1 gate for `./internal/fabricengine` passes (failed 0).

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: the F3 comment named the shared hub as the reason for subtests that never touch it. Fixed by the F3 change above.
- fix-consistency, LOW: row (8) kept "not the pre-flight's" and "the pre-flight itself" twins. Fixed: both name `checkResetTarget`.
- fix-proposal, LOW: same F3 comment finding as fix-consistency. Merged and fixed with it.

Both fixes are in commit "fabricengine: fix self-check findings — parallelise the two read-only lifecycle subtests, name checkResetTarget in the whole neutering row".
The lens reports are saved beside this file as `round-2-fixer-report.lens-<lens>.md`. No fork returned without a report.

## Disputed

None.

## Deferred

None. F1's edit exists in the working tree only, for the `_lym` reason above.
