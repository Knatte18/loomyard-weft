---
round: 1
focus:
  - "Fidelity to the decision record: check that each of D1-D9 and each Constraint is carried by a card, in particular D3 (no reconcile in Land), D6's exclusions (leased update and both deletes do not retry), D8's stencil edits, D9's `CLAUDE.md` and refusal-spec rows, and the Constraint doc list (battenshed, landingshed, fabricengine and gitrepo docs, the batten recipe header); also that no card adds scope the record does not license."
  - "Card 8 and card 10 (the largest cards): settle whether each step and each refusal path in their `**Intent:**` has a named input and a named test that reaches it (pause during the dirty-prime and land-lock waits, a spent prime-lock wait, `ErrChildNotParked` and `ErrChildDriverBusy` retries, a goto refused for an unknown target), and whether a test only calls an error constructor."
  - "Attack-surface question: for the deploy command (cards 5 and 8), the automatic Publish goto and resume (card 10), and the push retry that now covers a permanent remote `cannot lock ref` (cards 1 and 2), settle whether each card's `**Intent:**` states what it can skip or let through and what bounds it."
  - "Edit-card glyphs: on the `Edit` cards that change existing functions, types or fakes (cards 2, 3, 4, 7, 9, 10), settle whether the targets are member glyphs rather than bare file paths, and whether the re-sign arrow is present wherever the Intent changes a parameter, result, type parameter or receiver (for example `NewInnerRun`, `battenCLI.wire`, `childSpawnError`)."
  - "Card order across the 1-to-11 sequence: settle whether any card uses a symbol a later card creates (card 7's row notifier against card 8's `rowNotifier.ready`, `runMarkerSuffixes` created or edited in several cards, card 9 wiring card 8's producer and card 5's `deploy` key, card 10 using card 4's `ShedBusyKind` and card 3's record fields)."
  - "Test economy and the budget: settle whether any new test function repeats behavior an existing test already covers, or exists per symbol, across cards 1, 2, 6, 7, 9 and 10, and whether each card adding top-level tests lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`."
  - "Granularity: settle whether cards 3, 4 and 5 are each an independently reviewable unit, and whether the Create lists on cards 8 and 10 bundle private helpers with their type or split them into per-symbol targets."
  - "Verify coverage: settle whether the `## verify:` section runs the tests of every package any card targets (including `cmd/lyx`, `internal/testkit/envkit`, the nested-module question if any) and follows the plan template's section order."
  - "Quality of each `ImpactSummary` on `Edit` and `Delete` cards: settle whether it is a real blast-radius conclusion rather than a restatement of `Intent`."
---

Do-not-flag checks applied: no focus entry re-derives a validation-check ID, a missing `DependsOn`/`Produces` field, or an `ImpactSummary` on a non-Edit, non-Delete card.
`support-log.md` was not read.
