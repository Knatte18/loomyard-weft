# Fixer-report — code round 1

All six findings fixed; the self-check's findings are fixed too except one deferred. `lym gate test ./internal/landingshed` passes (tier0, no failures).

## Changes

- **F1** fixed as proposed.
  `landUnderLock` builds `MergeOptions.Squash` as `Config.Squash || landing.atOnce`; the retry merge reuses the same options.
  Rows: the merge-parent no-record row with `squashOff` now asserts every parent-side merge squashed (`wantSquashedMerges`), and a reopen row does the same.
- **F2** fixed as proposed.
  New row `AReopenedPullRequestUnderAMergeParentSquashMergesWhateverTheSquashSetting`: merge parent, record with the operator's ending, closed, `squashOff`; expects `Reopen 7` and the squash merge call.
- **F3** fixed as proposed.
  The two merge-parent open-path rows read the record back and compare Number, URL, Base, Head, GatedHead and ReviewEnded (`wantRecord`).
- **F4** fixed with the first option the review offered.
  `pullRequestLanding` reopens a closed recorded pull request under `MergeAtOnce` instead of halting.
  New row `AMergeParentReopensAClosedRecordedPullRequestWithNoOperatorEnding`.
  The `Call` doc, the `pullRequestLanding` comment and `doc.go` state the exception.
- **F5** fixed as proposed, by threading `landing.atOnce` into `pushParent` as `parentMerges`.
  `protectedParentWayForward` takes it and leaves out the `pull_request=merge` advice when set; the hand-landing steps stay.
  New row `AProtectedParentPushRefusalUnderAMergeParentOmitsTheSettingAdvice`; the two existing callers in `finalize_test.go` pass `false`.
- **F6** fixed as proposed: the `prLanding.merge` comment now says it merges the run's pull request and opens it first when `open` is set.
- **Self-check fixes** (see below): the merged-pull-request return now carries `atOnce`, with a new row `AMergedPullRequestUnderAMergeParentLandsThroughASquashingParentSideMerge`; `doc.go` lines on the protected-branch way forward, the closed-record halt, the reopen and the squash now match the code.

## Disputed

None.

## Deferred

- **Self-check, fix-proposal LOW — a deliberately closed pull request is now reopened under a merge parent (F4's change).**
  Whether a close on GitHub should stop a no-review landing, with a halt naming reopen and `lym batten cancel`, or be reopened and landed hands-free, is an operator decision on a real tradeoff.
  Under merge mode `PR-Review` writes no ending, so the record cannot tell a deliberate close from a stray one.
  I kept the reopen, the review's first-named option; switching to the halt is the review's second option, `finalize_pr.go:206` plus the halt text.

## Self-check

Forks: fix-code-correctness, fix-consistency, fix-proposal, fix-affected-set; all four returned a report. Reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, LOW: `atOnce` lost on a merged recorded pull request. Fixed as proposed (merged return carries `atOnce`; new row).
- fix-consistency, MEDIUM: `doc.go` closed-record halt and reopen lines stale after F4. Fixed as proposed.
- fix-consistency, LOW: `doc.go` protected-branch paragraph stale after F5. Fixed as proposed.
- fix-consistency, LOW: `doc.go` squash line silent on the parent-side merge. Fixed as proposed.
- fix-proposal, MEDIUM: F1 fix misses the merged re-entry. Duplicate of the correctness LOW; fixed with it.
- fix-proposal, LOW: F5 proxy false on the merged re-entry. Duplicate; fixed with it.
- fix-proposal, LOW: deliberate close reopened under a merge parent. Deferred, above.
- fix-affected-set, LOW: `doc.go` closed-record halt stale. Duplicate of the consistency MEDIUM; fixed with it.
- fix-affected-set, LOW: `doc.go` squash line. Duplicate; fixed with it.
- fix-affected-set, LOW: protected-branch doc text unchecked. Duplicate of the consistency LOW; fixed with it.

## Commits

The first commit holds F1 to F6 together rather than one per finding, since the findings share the same test table and call sites and the gate was run once over the whole set.
The second holds the self-check fixes.
