# Round 1 fixer-report

## Changes

- F1 (MEDIUM), fixed as proposed, commit `959b22c3`.
  `TestSamplerSession` in `internal/loomshed/samplersession_test.go` gains two rows whose `onWake` raises the done flag at wake 2 on a live session.
  The dirty-checkout row wants a cancelled-context error, one mail naming `main.go` sent to the resumed strand, a resume of the recorded session id and a record `DoneStamp` of 0.
  The damaged-record row wants Stuck with Halt and a reason holding the finding and `git -C _lym checkout`.
- F2 (LOW), fixed as proposed, commit `be364819`.
  In `decisionVerb` (`internal/loomcli/decision.go`) a failing `samplerStrand` read falls through to the parent comparison.
  It refuses with the "read the run's sampler session" message only when the caller is not the run's parent, or when the parent cannot be resolved either.
  `TestDecisionVerb_AppendsAndCommits` gains a `samplerErr` field and a row for the recorded parent with an unreadable sampler record, which wants the append to succeed.

`lym gate test ./internal/loomshed ./internal/loomcli --tier 0` passed after both fixes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; it left the wake-path error branches (`DirtyPaths`, `CheckRecord`, `stopLive` failing) unchecked, which the review did not name.

## Disputed

None.

## Deferred

None.
