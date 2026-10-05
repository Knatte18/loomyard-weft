# Round 1 fixer report

## Fixed

- **F1 (D4 voucher lifetime):** `decision-record.md` D4's Bound now states that start writes the voucher only after its refusals pass, right before the spawn, replacing any earlier one, and only when Tier 2 is on.
  It also states the stale-voucher case and its bound (at most one later matching observation, until the next start or completed step overwrites it) and that the parked-driver resume path writes no voucher because its run is halted.
- **F2 (D5 bound):** D5 now carries a Bound: the drop rule can leave a real lyx fault unfiled, and the note's trace file, the run's status and history and `lyx selfreport create` keep the evidence and the manual route open.
- **F3 (D2 loader question):** D2 states unconditionally that the stale `selfreport` key loads and the loader needs no change.
  The matching Open risk is deleted, the plan-writer note says only a pinning test is needed, and `support-log.md` Q8 is resolved.
- **F4 (two homes):** D1's Decision no longer lists identifiers; Scope-In is the one list of removals.
  The crash-resume acceptance criterion points at D3's field list instead of repeating it.

## Deferred

None.
