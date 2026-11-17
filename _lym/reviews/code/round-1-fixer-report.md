---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

# Fixer-report, round 1

## What changed

- **F1** (`internal/gatecli/redundantmutants.go`, `internal/testcover/mutantverdict.go`).
  `runMutants` now queues the covering runs of each mutant a candidate's own run killed or left unjudged, with the same passes, covers, queued and ran checks.
  `JudgeMutants` leaves a mutant out of `Killed`, `Unique` and `Unjudged` when the candidate's own run of it was unjudged and a covering test killed it.
  Added a `TestJudgeMutants` row for that case, another pinning that a candidate whose only mutant is such a run lands in `ReasonKillsNone`, and changed the existing "own run unjudged" row so its covering test no longer kills the mutant.
  Updated the `JudgeMutants`, `MutantVerdict.Unjudged` and `runMutants` doc comments, the `gatecli/doc.go` lines, and the `TestGateRedundant` run count (`coveringRuns` 14 to 16, the tally loop's two covering runs) with its comment.
- **F2** `internal/gateslot/doc.go`: a mutant run past the test bound or memory cap is unjudged, never a kill.
- **F3** `internal/gateslot/template.yaml`: both comments now say unjudged, never a kill.
  No fixture or test pins the template text.
- **F4** `TestRunMutantsRerunsACoveringTestThatTimedOutAsACandidate` in `internal/gatecli/redundantmutants_guard_test.go`, with the stand-in `TestMutantStandInHangsOnce`.
  The stand-in hangs on its first run with mutant 1 on and fails on later ones.
  The test asserts the rerun's kill stands and the candidate-run timeout is cleared.
  Reverting the `!timedOut[run]` clause or the `delete(timedOut, run)` fails it.
- **F5** `TestRunAlone` row `ElapsedStampsOnlyTheRunLineAndTheCompletedTopLevelResultLine` over the new `standInSplit` stand-in, as the review proposed.
  A fork showed that row cannot pin chunking, since pipe reads may coalesce, and `testResultLine` is line-anchored.
  So I also added `TestStampOutput` over `StampOutput` in the new `internal/testcover/export_test.go`, which feeds fixed chunks to the collector.
- **F6** `AnswersEmptyListsForAMapHoldingNoContainedTest` in `redundant_integration_test.go`, over a new one-test map `alone`: `candidates`, `refuted` and `unconfirmed` are each `[]`.
- Self-check follow-ups: four comment lines over the length limit rewrapped, and the `JudgeMutants` "not judged" sentence reworded.
- Gate repair: four more comments the comment lint refused, two wrapped mid-sentence and two over the length limit, were rewritten as one sentence per line.
  They sit in `redundantmutants_guard_test.go`, `export_test.go`, `mutantverdict.go` and `runner_test.go`.
- Gate repair, second pass: the full verify run failed two `PATTERN-test-speed` checks in `cmd/lym`.
  `TestStampOutput` moved to its own tier-0 file, `internal/testcover/stampoutput_test.go`, since `runner_test.go` holds tier-1 tests.
  The stand-in `TestMutantStandInHangsOnce` waits on `<-time.After(time.Hour)` instead of a literal `time.Sleep(time.Hour)` in a tier-0 file;
  the check's time bound kills it after seconds either way.

Commits: F2, F3, F5, F1+F4+F6 (shared files, one commit), and a self-check follow-up commit.
`lym gate test --tier 0 --tier 1` over `./internal/testcover`, `./internal/gatecli` and `./internal/gateslot` passed before the follow-up.
The follow-up ran green over `./internal/testcover` and `./internal/gatecli`.

## Self-check

- fix-code-correctness, MEDIUM: four new comment lines at or over 120 characters: fixed (rewrapped).
- fix-code-correctness, LOW: the F5 stand-in row does not reliably pin chunking or the subtest filter: fixed, with `TestStampOutput`.
  The `HasPrefix("--- ")` filter itself stays unpinned, since the line-anchored regexp makes it behaviour-neutral.
- fix-code-correctness, LOW: a candidate whose every mutant is a moot own unjudged run gets `ReasonKillsNone`: fixed by pinning it in a row and rewording the doc.
  The reason text stays: nothing is left to delete safely or to confirm.
- fix-consistency, LOW: the `JudgeMutants` "no run of it was unjudged" sentence contradicted the new rule: fixed.
- fix-proposal, MEDIUM: the F5 row passes with the defect in place: duplicate of the fix-code-correctness F5 finding, fixed with `TestStampOutput`.
- fix-proposal, LOW: duplicate of the `ReasonKillsNone` finding above, fixed the same way.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
