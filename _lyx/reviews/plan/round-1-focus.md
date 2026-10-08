---
round: 1
focus:
  - "Cards 5 and 6 retire guards (`containment-unit-overlap`, `containment-file-overlap`, `handle-unreferenced`): does each card's `**Intent:**` state what the removal lets through and what bounds it, as decision D2 and D4 do?"
  - "Cards 8, 10 and 11 add plan-gate-only checks beside every-gate checks: does each card keep the check out of `ValidateDispatch` and state the bound for what dispatch lets through, and does each reach `lyx loom validate-plan` through the same function (`PATTERN-gate-self-check-parity`)?"
  - "Constraint coverage in `decision-record.md`: does some card carry the `PATTERN-ref-shape-registry` ledger entries and gate constants for the new checks and the removal of retired gates, the `TestChokepointCallSites_PinnedToTheirGuardedFunctions` pin (no new `quarry.Name` or `Resolve` call site), and a spec row plus a test reaching each new blocking finding?"
  - "Test economy across cards 1-11: do the new rows in `glyphchain_test.go` and `glyphchain_integration_test.go` duplicate behavior that an existing test in `planparser` or `planglyph` already covers, or add one test per symbol rather than per public-surface behavior?"
  - "Card order: do cards 9-11 depend only on earlier cards (`plan:internal/planparser#CardResign` from card 9), and do the D2/D4/D3/D5/D6 flips named in D7 (`rework-generations` row, containment rows, `handle-unreferenced` rows) land in the card the record assigns?"
  - "Card 11 granularity and scope: the check and its fixture module files share one card; settle whether the `deleteorder.go` helper extraction and the fixture files are licensed by D5 and D1, and whether any listed fixture or behavior goes beyond the record."
  - "`## verify:`: does it run the tests of every package a card targets (`internal/planparser`, `internal/planglyph`, `contracts/stencils`, including the `integration` tier), and does it follow the order the plan template states?"
---

Focus entries name where to look; none caps severity or pre-states a verdict.
Mechanical checks, derived `DependsOn`/`Produces` edges and missing `ImpactSummary` on non-`Edit`/`Delete` cards are left to the gate.
