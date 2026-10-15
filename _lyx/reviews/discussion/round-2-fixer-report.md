# Round 2 fixer-report — discussion

## Changes

- F1 (MEDIUM, design), fixed as proposed, with one clause kept.
  Premise confirmed: `evaluateGate` in `internal/shuttleengine/wait.go` runs every entry's closure in list order on each evaluation, and `parent-review` is the last, `MayHold` entry (`internal/shedrecipe/entries_gate.go`).
  D5's held-entry sentence now covers every entry of the gate: while an entry is held pending, the re-run evaluations record no gate span for any entry.
  The sentence keeps its closing "no two spans of one entry overlap", since D6 cites it.
  D6 adds that an entry listed before a held one shows only the evaluations outside the hold.
  The gated-run acceptance criterion adds that an entry listed before a held entry reports no gate span for the re-evaluations during the hold.
- F2 (NIT, consistency), fixed as proposed: the support log's Turn 3 goal-scope bullet now cites the dotted `6.5` form, matching D4.

## Disputed

None.

## Deferred

None.
