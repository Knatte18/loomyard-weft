---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` (D1 to D10, plus acceptance lists) against the cards' Intents; settle whether each is carried by a card and whether any card (notably 4, 8 and 9) adds scope the record does not license."
  - "Card 9 and card 7: the card that authorizes forks under `twill` and the card that removes the ready marker before a fresh Plan-Write spawn each introduce something that can skip or let through; settle whether each `**Intent:**` states what it can skip and what bounds it."
  - "Reachability: for every refusal or guard branch a card prescribes (cards 2, 3, 4, 6, 8, 9), settle whether a named input and a named test reach it, and whether an upstream check already rejects that input."
  - "Test economy: cards 3 and 4 add new test functions (`TestIndex_AppendsFabricVocabularyName`, `TestDirective_MissingValueNamesTheMarker`) and card 2 a table test; settle whether an existing test in `planglyph`, `twilldirective` or `planparser` already covers the behavior."
  - "Edit glyphs and the re-sign arrow: for each `Edit` card (3, 4, 6, 7, 8, 9), settle whether Go file targets with Intent naming symbols should list member glyphs, and whether any member whose parameters, results, type or receiver the Intent changes lacks the re-sign arrow (for example `Values.ReviewedFiles` changing type is a struct field and takes none)."
  - "Card order and dependency: cards 2 and 3 use symbols created in cards 1 and 2, and card 9 uses cards 4 to 8; settle whether any card uses something a later card creates."
  - "Verify: settle whether every package any card targets, including `contracts/stencils`, `internal/loomengine`, `internal/loomcli` and `cmd/lyx`, has its tests run by the `## verify:` section, and whether the section is in the plan template's order."
  - "Granularity: settle whether card 1 (a package plus its two callers' edits) and card 9 (the largest card) are one independently reviewable unit each."
---

Focus entries were checked against the rubric's `Do not flag` list; none re-derives a gate check or a derived dependency field.
