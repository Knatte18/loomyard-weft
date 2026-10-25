# Fixer report — plan, round 2

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Premise confirmed: `HistoryEntry.Reason` holds only a skipped entry's gate text (`internal/shedengine/status.go:118`), and the step-3b resume write persists `StateRunning` with an empty error before the producer runs (`internal/shedengine/run.go:296`), so a resumed entry cannot read a Stuck reason.
  - `_lyx/plan/02-drift-records.md`: `ScopeRecord` gains `gate_halt` (the opening text of the dirty-tree or test-gate Stuck `Drift-Verify` last returned) and `gate_halt_at` (when).
  - `_lyx/plan/11-drift-verify-producer.md`, step 7: the reason-text test is replaced.
    Each step-7 halt and each step-10 exhausted-entry Stuck writes both fields through `UpdateScope` before returning.
    An entry is the resume when `gate_halt_at` is set and the last history entry is a Stuck of this row at or after it, the same test as step 3.
    Such an entry clears both fields once it passes the dirty check, then runs the told `verify` entry's gate once, only when `AttemptHead` is found and differs from HEAD.
    A gate failure writes the fields again.
    The opening texts stay package constants that open the reasons for the operator; the step now says no code reads them back.
    One small departure from the proposal: the fields are cleared on passing the dirty check whether or not the gate then runs, so an entry with `AttemptHead` equal to HEAD doesn't leave a stale mark behind.
  - Step 10: the exhausted-entry Stuck writes `gate_halt` and `gate_halt_at` as step 7 says.
  - `TestDriftVerify`: the halts record both fields.
    The resume rows seed `gate_halt_at` and a last history entry that is a Stuck of this row, not a reason text.
    New rows: a passing resume gate clears the fields, and a last history entry of another row, or one older than `gate_halt_at`, runs no gate.
  - Card 11's **Uses** gains `plan:internal/driftengine#UpdateScope`.

## Self-check

- fix-proposal, MEDIUM: the mark goes stale after a goto, so a later Stuck of the row at another step passes the history test and runs the gate on commits from a different row.
  **Fixed with own change.**
  The fork proposed clearing the fields at step 7 on a non-resume entry.
  I didn't make that change, because the fork's own case has the re-entered row halt at step 5, before step 7, so the stale mark would survive.
  Instead, step 7 says every entry decides whether it is a resume once, right after step 2, and a non-resume entry clears both fields there, before any halt.
  That also covers an interrupt between the `UpdateScope` write and the shed persisting the Stuck.
  A `TestDriftVerify` row covers it: after a goto, the next entry clears the mark, and resuming that entry's no-counted-capture halt runs no gate.
- fix-consistency, LOW: after "past any `DecisionHaltPrefix`" was dropped, nothing said how the prefix and a halt's opening text are ordered.
  **Fixed as proposed**: step 7 now says the opening texts open the reasons "after `DecisionHaltPrefix` where it applies".
- fix-claims: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
