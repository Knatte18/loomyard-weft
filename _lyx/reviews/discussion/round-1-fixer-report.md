# Fixer report: discussion round 1

## Fixed

- **F1 (BLOCKING):** D8 now defines "another ref" for a remote tip as excluding the branch's local ref and every remote-tracking ref of the same name, with the loss path it prevents; D9 inherits the exclusion explicitly.
  Added an acceptance criterion: an unlanded pushed warp branch keeps its origin copy under `remove --remote`.
- **F2 (BLOCKING):** D6 now runs the outside-pathspec sibling-dirtiness refusal with the other refusals, before its commit; D3's probe includes that check, and D3's guarantee is narrowed to calls the probe refuses, naming the probe-to-`Remove` race.
  Added an acceptance criterion for the sibling-side refusal.
- **F3 (MEDIUM):** D7's bound restated: the mark records a removal request from any `reed remove --detach` caller, does not prove a successful commit (uncommitted records stay for D6 or the next commit), and is cleared when the remover spawn fails.
  D2's rationale notes that a failed commit is caught by D6.
  Support log Q14 records the clear-on-spawn-failure pick.
- **F4 (LOW):** D2's bound states that a parked driver is ended without refusal and why that loses nothing.
- **F5 (LOW):** D5's not-found error names a stray path at the pair location; the matching acceptance criterion covers it.
- **F6 (LOW):** The warp-SHA pick moved from Open risks into D6; the risk is gone and Q13 is resolved.
- **F7 (LOW):** The Goal now says no landed branch is left locally or, with `--remote`, on origin.
- **F8 (NIT):** D10 lost its dated verification sentence and board-item history; the decision and its invariant-based rationale remain, and the support log's Interview turn 11 keeps the check.

`lyx loom validate-discussion` exits 0 after the edits.

## Deferred

None.
