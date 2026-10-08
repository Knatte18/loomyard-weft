# Fixer report — discussion round 1

Every finding was validated against the code and fixed.

## Fixed

- **F1 (BLOCKING), D6 re-prompt retry never fires.**
  Confirmed in `wait.go`: the per-tick re-entry is gated on `run.gatePending && run.gateAtBoundary`, and a failed re-prompt is not pending.
  D6 now marks the boundary as holding an unsent re-prompt, and that per-tick branch re-enters `handleGatedBoundary` for the mark, re-evaluating the gate and re-sending without a new writer event.
  A writer turn starting meanwhile clears the mark.
  In-loop sends cap their idle wait and submit window at the time left until the run deadline, overshooting only by the fixed appearance check.
  The Bound now states why a `turn_start`-cleared boundary is progress, not a loop.
  Acceptance criterion reworded to pin the retry on a later tick with no new writer event; Notes pointer updated to the per-tick branch.
- **F2 (MEDIUM), `stopped` outside the file-contract-first check.**
  Confirmed in `attach.go`'s `dispositionCandidate`: the check fires only for `runOutcomeRunning`.
  D7's stop verb now consults `allOutputFilesExist` first and leaves a `running` record whose outputs all exist untouched, removing only the strand, so attach's existing branch harvests it.
  The `PATTERN-completion-signal` Constraint is reworded: the stop verb is a new negative-answer site whose file and sites join the tripwire in `completionsignal_enforcement_test.go` in the same commit.
  Rejected alternative (writing `done`) and auto-pick Q10 recorded in the support log.
- **F3 (MEDIUM), unmatched `turn_start` holds a send against an idle pane.**
  D1 now releases the hold when `TurnStartInterrupt` reports the turn interrupted, or after 10 s of idle pane, with a Warn; the Bound states the 10 s cap.
  The Open risks bullet now separates a missing `UserPromptSubmit` line from a missing `Stop` line.
  Acceptance criterion extended; auto-pick Q9 and a rejected alternative recorded.
- **F4 (MEDIUM), D4's "no work done" premise unbounded.**
  D4's Bound now states that `ErrSessionBusy` types nothing, that start re-reads the box and signals after `ErrSubmissionNotLanded` and starts the run if the prompt landed late, and names the residual gap with the re-step's `Spec.validate` refusal.
  Acceptance criterion added for the late-landed prompt.
- **F5 (MEDIUM), D2 settle vs D3 window.**
  The submit window now opens when typing begins and covers D2's settle and D3's confirmation; a box that never settles gets no Enter and fails with `ErrSubmissionNotLanded`.
  D2, D3 and the settle acceptance criterion updated.
- **F6 (MEDIUM), `Wait` overwrites `stopped`.**
  Confirmed: `finalize` writes `run.state.Outcome` unconditionally.
  D7 now has the stop verb mark the handle under its lock, so that handle's own `Wait` persists `stopped` for every outcome but `done`; only a `Wait` in another process may still overwrite.
  Acceptance criterion added.
- **F7 (LOW), failed record write.**
  D7: a failed record write returns the error and removes nothing; the stop-verb criterion pins it.
- **F8 (LOW), acceptance gaps and unnamed outcome.**
  D4 now names `died`.
  Criteria added for the guid-only stop, the already-terminal record, `Runner.Send` waiting, the prompt-offset persist teardown, the 15-line cap and failed-capture message, and per-key zero/negative config errors.
- **F9 (LOW), Q8 open while D4 reads as decided.**
  Open risks now states that D4's class widening is an auto-pick awaiting the operator (Q8); Q8 stays open since that choice is the operator's.
- **F10 (NIT), restated content.**
  Dropped the defaults bullet from Auto-mode assumptions and the incident-chain item from Notes for the plan writer; the code pointers stay.

The support log's Review rounds section records this round.

## Disputed

None.

## Deferred

None.
