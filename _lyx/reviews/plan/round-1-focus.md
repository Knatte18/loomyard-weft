---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` and settle which card carries it; also settle whether any card (cards 11-13 on `OrElse`/`Exit` and `lyx shuttle open`, card 10 seat groups) introduces scope the record does not license."
  - "Card order: cards 8-9 use the `Present` and `OwnWindow` fields of `reedengine.StrandStatus` that card 3 adds, and card 9 and card 13 depend on cards 4-7 and 11-12; settle whether any card depends on something a later card creates."
  - "Attack surface: cards 3 (ends orphan own windows by `@lyx_own`), 6 (removal moves a viewer) and 13 (new `lyx shuttle open` verb) add a reap, a removal path and a verb; settle whether each card's `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Reachability: for every refusal or guard branch a card prescribes (`window_split` validation in card 2, negative `windowSize` in card 1, the refusal paths of card 13), settle whether a named input and a named test reach it and whether an upstream check already rejects that input."
  - "Edit glyphs: for Edit cards whose Intent names specific symbols or changes signatures (cards 3, 4, 5, 8, 9), settle whether member glyphs are listed and whether changed signatures carry the re-sign arrow; card 8 lists `internal/shuttleengine/doc.go` and a Delete of `removeStrandIfLive` to check against its Intent."
  - "Test economy and granularity: settle whether any card plans a new test for behavior an existing test already covers, or one test per symbol, naming the covering test; settle whether cards 8 and 10 bundle or split units correctly."
  - "Verify coverage: settle whether every package the cards target (render, reedengine, shuttleengine, the seat engine, the provider seam, the CLI package of `shuttle open`) has its tests run by `## verify:`, and that the section order matches the plan template."
---

Seed focus from a read of `00-overview.md` and the card headers; the lens file `lens-review.md` and `support-log.md` were not used.
