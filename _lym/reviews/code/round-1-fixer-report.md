# Fixer-report, round 1

## Changes

- **F1** fixed as proposed (2a8ad1f8): `runTurnStarts` and `operatorInstructions` in `internal/shuttleengine/carryover.go` state that a turn start the events file holds no Raw line for is not read.
  The wording was tightened in the follow-up commit after the fix-proposal lens noted that an empty Raw is dropped too.
- **F2** fixed as proposed (49359e1b): a row in `TestWait_HeldTurnEndNotice` runs a `HoldDeferral` run with an operator-typed turn and expects no notice and `NotifiedOffset` 0.
  The follow-up commit corrected the row's name.
  `lym gate test ./internal/shuttleengine` passed after the row was added.
- **F3** disputed, see below; the change I first made (b54430fd) is reverted (3df327be).
- Follow-up commit, own change from the self-check: `doc.go` line 460 said the run's prompt is "the first" turn start, which `runTurnStarts` no longer holds; it now says "the turn start of the run's prompt".

## Disputed

- **F3, premise false.** The review says `doc.go` holds no carry-over or sent-texts paragraph below line 109.
  `doc.go:459-460` is that paragraph ("Operator instructions carried over: ... the run directory's sent-texts file"), so "as the carry-over below reads one" resolves.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW (F1 wording omits an empty Raw): fixed as proposed in the follow-up commit.
- fix-affected-set, LOW (`doc.go:460` says the prompt is "the first" turn start): fixed with the follow-up commit's `doc.go` edit.

The follow-up commit's edits are wording only and were not re-run through the gate.
