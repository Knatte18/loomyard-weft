# Fixer report — plan, round 2

## Changes

- **F1** (fixed as proposed): card 10's spawn-path table gains a row for a status file that exists but does not parse or carries an invalid state, among the refusals that spawn nothing, write no attempt directory and assert their way forward.
- **F2** (fixed as proposed, first option): card 10's refusal-spec sentence classes `arbiter: status or envelope unreadable` as a correctness halt, matching the shed section's `status file missing` row, and leaves raw I/O failures on the section's grouped transient row.
- **F3** (fixed as proposed): card 3 moves the flag-without-`--until-stop` case into `TestLoop_RefusalsBeforeAnyStep`, now four rows (asserting `kind: bootstrap`, no child, the hook never called, `arbiter_attempt`); `TestLoop_ArbiterAttemptConsumedByTheWaiter` covers only the hook-returns-nil path.
  The test budget's raise of one is unchanged, since the new test remains.
- **F4** (fixed as proposed): card 14's `Bound:` gains D2's half for a driver running the verb at a `producer`, `bootstrap` or reinvoke `interrupted` error without a fork escalation, bounded by the spawn budget, the applied-attempt refusal and the arbiter having no repair verbs.
- **F5** (fixed as proposed): card 10 step 2 gains a branch before `otherwise hand-on, cause missing`: with no marker for the attempt and its recorded deadline ahead, the verb probes `AttachIfLive` with the attempt's spec and `Timeout` the time left to that deadline, waiting on a found run as the live-marker branch does and stopping it through `StopStrand` with the guid `attempt.json` records.
  The re-run-path table gains a row for an attempt with no marker whose strand is live.

## Disputed

None.

## Deferred

None.
