---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` (D1 to D9 and the constraints) and settle whether some card carries each, and whether any card, notably card 6's fake-runner gate-once change and card 5's added mode-rules sentence, adds scope the record does not license."
  - "Card order: settle whether card 6 depends on anything in cards 7 or 8, whether card 5 depends on a symbol a later card creates (the `Prompts` seam of card 1, the stencils of card 4, the config of card 3), and whether card 7 deletes anything card 6 still references."
  - "Card 6 granularity and size: it batches the Env change, the recipe entry, two fakes, loomrecipe fixtures, loomcli wiring and an integration test; settle whether it is one independently reviewable unit or should split along the existing symbol boundaries, and whether the `shedfake.SeatRunner` gate-once change is a public surface that needs its own card."
  - "Test economy: for the tests cards 2, 3, 5 and 6 plan as new functions or rows, settle whether an existing test already covers the behavior; name that test, and check the card 5 and card 7 budget-file trajectory for consistency with the new top-level tests."
  - "Re-sign arrows: on each `Edit` member glyph in cards 2, 3, 5 and 6 whose `Intent` changes parameters, results or receiver (for example `MultiLLMProducer.Call`, `loomCLI.wire`, `engineMinimalConfig`, `SeatRunner.Run`), settle whether the arrow is present; struct fields and interface methods take none."
  - "Attack surface: for the fresh-spawn preparation hook (card 2), the optional-values list (card 1), the gate wrapping and nil-preparation rule (card 6) and the advisor-config validation (card 3), settle whether each `Intent` states what the change can skip or let through and what bounds it."
  - "Verify coverage: settle whether the `## verify:` commands run the tests of every package the cards target, including `internal/testkit/...` and `cmd/lyx`, and whether the section follows the stencil's order."
---

Seed focus from a read of the overview and cards 2, 5, 6 and 7; the remaining cards are left to the round's own reading.
