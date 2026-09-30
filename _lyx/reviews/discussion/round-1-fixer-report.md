# Round 1 fixer report — discussion

All eight findings fixed, in `_lyx/discussion/decision-record.md` unless noted.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1 (MEDIUM), clearing and resuming unbounded:** D10 now bounds every non-idle phase by the handoff timeout from its persisted entry time.
  `clearing` waits for D9's idle-session probe instead of the startup-ready classification, and moves to `resuming` when the probe passes or the timeout expires.
  `resuming` returns to `idle` at the first turn end after the injection, or on timeout with an abort reason.
  Per-phase restart rules replace the general "never re-injects" sentence, with the rationale that a repeated `/clear` after a completed handoff and a resume prompt in an uncleared session are both harmless.
  Two unit-test scenarios added to Acceptance criteria.
- **F2 (MEDIUM), stale phase into a fresh session:** D10 records the owning strand with the phase, and a fresh launch resets the phase to `idle`, recording the abandoned phase.
  D14's `stop` points at that rule, and a unit-test scenario was added.
- **F3 (MEDIUM), aborted handoff as resume source:** D10 records the last completed handoff when `handoff-requested` completes, and D6's second source is that recorded file.
  The rationale explains why the newest file does not qualify.
  The Auto-mode assumptions and the Acceptance criteria wording follow, and a unit-test scenario was added.
- **F4 (MEDIUM), stale pre-clear usage reading:** D7 says a reading comes from the transcript named by the turn end being evaluated, and only turn ends after the last injection are evaluated.
  A unit-test scenario was added.
- **F5 (MEDIUM), turn in progress invisible:** D9's probe now classifies an idle session, which adds "no turn in progress", and the rationale says why the events file cannot see a turn start.
  The probe's fixture list gains a turn-in-progress capture.
- **F6 (LOW), "focus line":** D10 says the focus text follows the path on the same line, and Open risks says "focus text".
- **F7 (LOW), compaction margin:** The last Open risks bullet now states that operator turns before idle and the handoff turn spend the default's margin on a 200k window.
- **F8 (LOW), background-task survival:** The smoke test starts a background task before the cycle and records in the module doc whether it and its completion notification survive `/clear`.
  Open risk 1 points at this step.

In `_lyx/discussion/support-log.md`: Q13 now names the smoke-test verification, and Q14–Q16 record the three questions this round resolved.
Q11 and the auto-relaunch rejected alternative now say "last completed handoff".

## Deferred

None.
