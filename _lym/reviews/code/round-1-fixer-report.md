```yaml
round: 1
fixed: [F1, F2, F3, F4, F5]
```

## Changes

- **F1**, `internal/fabricengine/handover.go`: fixed with my own change.
  `pushOwedCodeTip` now pushes the owed tip when origin has no code branch, creating the branch with a plain non-forcing push.
  The proposed absent-branch lease cannot be made: `UpdateRemoteBranchLeased` rejects an empty lease as `ErrInvalidSHA`.
  When the last transition no longer names the tip, `deleteGivenUpCodeBranch` saves the tip under its owner archive tag, then deletes the branch it created under a lease on the tip.
  Docs of `pushOwedCodeTip`, `HandOverTips` and `HandOverResult.TipPushed` follow.
  Two `TestHandOverTips` rows cover an absent origin branch: the tip is pushed, and a given-up tip leaves the branch deleted with the tag on origin.
- **F2**, `internal/beamcli/deps.go`, `internal/beamcli/pr.go`: fixed with my own change.
  `PushBeam` pushes through `pairteardown.PushRun`, which carries `Here` and the run slug, with `InsidePair: true`.
  A beam verb may run inside the pair, so a hand-over leaves the pair for the parent's dobby to remove.
  `pushRefusal` returns a push that handed the run over unchanged, instead of telling the operator to re-run.
  A `TestPROpen` row pins that refusal.
  Proposed `PairPush` alone was not used: it would return the raw takeover error and hand nothing over.
- **F3**, `internal/landingshed/finalize.go`: fixed as proposed, the second of its two options.
  `pushParent` calls `recordLanding` again after `archiveHeldParentMerge` succeeds, so the record holds the reset branch's head.
  The archive row of `TestFinalize` now records the head before and after the archive and asserts the order.
- **F4**, `internal/landingshed/finalize_test.go`: fixed as proposed.
  The `//testtiming:keep` line sits above `TestFinalize_RecordsTheLandingCommitOnTheFirstTry`.
  The edit went in with the F3 commit, which touches the same file.
- **F5**, `internal/battencli/commitstatus_test.go`: fixed as proposed.
  The `//testtiming:keep` line sits above `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`.

Gate repair: a comment-format gate then refused the round.
`pushOwedCodeTip`'s doc had a line of 120 characters or more, now split into two sentences.
Two wrapped comments in `internal/loomcli/wiring_commitstatus_integration_test.go` are now one sentence per line.
No code changed.
A second gate then found those edits uncommitted, so they are committed as one comment-format commit.

Gate: `lym gate test` tier 0 over `fabricengine`, `landingshed`, `beamcli` and `battencli` passed, and tier 1 over `fabricengine` passed, with the new rows run.
Commits: one per finding, plus a follow-up for F2 after the self-check.

## Self-check

- fix-code-correctness, MEDIUM: `PushRun` removes the beam's pair when `InsidePair` is false. Fixed with my own change (`InsidePair: true`), see F2.
- fix-code-correctness, MEDIUM: no test of the beam takeover path. Fixed in part: `TestPROpen` pins the handed-over refusal. The hand-over itself stays deferred, see Deferred.
- fix-code-correctness, LOW: an absent origin branch is also what a deliberately deleted task branch looks like. Disputed, see Disputed.
- fix-consistency, LOW: `HandOverTips` and `HandOverResult.TipPushed` docs stale. Fixed.
- fix-consistency, LOW: `PushBeam` callers and `InsidePair`. Fixed with F2's follow-up.
- fix-consistency, LOW: no test for the beam takeover. Same as the deferred item.
- fix-proposal, MEDIUM: `HandOver` with `InsidePair` false removes the pair, and `pr.go` wraps a hand-over as a re-run. Fixed, see F2.
- fix-proposal, LOW: an absent branch may be a deleted task branch. Same as the disputed item.
- fix-affected-set, LOW: `InsidePair` left false. Fixed, see F2.

## Disputed

- Ground `premise false`, fix-code-correctness LOW and fix-proposal LOW, on F1: a run whose records hold a transition naming a tip and a takeover entry is a live run.
  Cleanup deletes the task branch only together with the run's records (`internal/fabricengine/cleanupremotewarp.go`), so no path shows a transition for a run whose branch was removed on purpose.
  The reading of an absent branch as a missing tip is the review's own premise (`OriginCodeTip`).

## Deferred

- The test of the beam hand-over through `PushBeam`: a diverged pair with another machine's takeover entry is built by fixtures in `fabricengine`'s external test package, which `beamcli` tests cannot import.
  Moving them is outside this round's write surface. The takeover read itself is pinned by `TestRejectedRecordsPush_ReadsTheTakeover` and `PushRun`'s own path.

## Rulings

None.
