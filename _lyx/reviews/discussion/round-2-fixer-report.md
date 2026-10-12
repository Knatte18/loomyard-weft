# Round 2 fixer-report: discussion

Each finding's premise was checked against the code before fixing:
`shedengine.Status` (`internal/shedengine/status.go`) has no reason field;
the bare and cancelled pauses in `internal/shedengine/run.go` return a `StepResult` with no `Reason`;
`Goto` (`internal/shedengine/goto.go`) persists `StatePaused`;
the publish-retry marker lives under batten's `scratchDir` with unexported read and write helpers (`internal/battenshed/publishretry.go`);
`lastStepOf` (`internal/shedverbs/steprecord.go`) compares against the calling `lyx`'s build.

## Changes

- F1 (fixed as proposed, option a): D13 rewritten.
  A reason lives only on the step result and envelope, with no status field added.
  `StepResult.Reason` is set to `pause requested` on the bare pause and to `cancelled` on a cancelled context in both `stepLocked` branches.
  The goto is dropped from the empty-reason list, and the decision now says the goto is no step stop.
  D12 gains the rule that its re-read classifies a `paused` status by its last history entry, with `OutcomeGoto` read as a moved run.
  The rationale, the Scope bullet and the D13 acceptance criterion follow.
- F2 (fixed as proposed): D4's retry count is now in two places, each written by its own actor and read by the other.
  Batten's `publishRetryDecision` reads a Go verify retry from the child's settle records.
  The Go driver reads batten's marker only through a `battenshed`-exported read taking the prime's scratch directory, and only when the batten-watched marker names a prime.
  The sentence saying the Go driver writes batten's marker is removed, and the retry acceptance criterion names the settle-record read.
- F3 (fixed as proposed, using the review's undecidable branch): D7 now runs D12's re-read first.
  History entries carry no actor, so a goto by the tackler and one by the orch are indistinguishable.
  For that reason every stop-key change goes to D12 as a new stop, with the verdict discarded and a stop-report line noting the change.
  `tampered` is kept for worktree changes made while the stop key is unchanged.
  An acceptance criterion covers an orch goto during a tackler wait.
- F4 (fixed as proposed): D7's spawn record and comparison now cover the stop key, HEAD, the checked-out branch, and the uncommitted paths with a content hash per path.
  The tampered acceptance criterion gains the commit, HEAD, branch and already-dirty-edit cases.
- F5 (fixed as proposed): D3's hand-over row now keys on the last step's recorded build differing from the on-disk build (D5).
  D5 says the Go driver reads the on-disk identity itself and never uses `binary_changed`.
- F6 (fixed as proposed): a tackler `goto` may name only the current row or the row the halting history entry routed from.
  This bound is stated in D7's verdict shape, its malformed list (cause `precondition`) and its bound sentence, with a row added to the malformed-case acceptance criterion.
  D4's Finalize sentence now says the tackler's goto stays within D7's bound.
- F7 (fixed as proposed): Q10 in `support-log.md` reworded to match D13.

## Disputed

None.

## Deferred

None.
