---
round: 1
focus:
  - "Fidelity: check that every Decision D1 to D15 and every item under `## Constraints` in `_lyx/discussion/decision-record.md` is carried by some card, and that no card adds scope the record puts under `Out`."
  - "Fidelity, D13 and D14: check that the Intent of cards 9, 10 and 11 states what the new check, the shared matcher seam and the read-only accept route can skip or let through, and what bounds each, as the record's `Bound:` paragraphs do."
  - "Attack surface, D4 and D5: check that cards 1 and 2 state what bounds a card that exceeds `budget` at its position and a batcher position that is too high or too low."
  - "Test economy: for the new tests in cards 5, 6, 9 and 11, check whether each plans a test for behavior an existing test already covers, or one test per symbol rather than per public surface. Name the covering test in any finding."
  - "Granularity: check that cards 5 and 6 (both in `tools/tokencount/calibrate.go`) and cards 9 and 10 each split along independently reviewable units, and that no card bundles a public symbol reused across packages into another card."
  - "Verify coverage: check that every package a card targets, `internal/batcher`, `internal/websterengine`, `internal/webstercli`, `internal/planparser`, the fabric-side matcher package and `tools/tokencount`, has its tests run by the `## verify:` section, and that the section follows the template's order."
  - "Card order, D12: check that each card's `Uses` only reach targets of earlier cards, and that cards 8, 9, 10 and 11 build the shared matcher before its consumers."
---

Focus entries name where to look; they set no severity and state no verdict.
None of them re-derives a check the plan gate already runs, a missing `DependsOn` or `Produces` field, or an `ImpactSummary` on a card type that does not require one.
`support-log.md` was not read.
