---
round: 1
focus:
  - "Card 1 granularity: do the Create glyphs for `EntryUser`/`EntryAssistant` and the `ToolShell`/`ToolFileRead`/`ToolOther` constants, or other private-surface-free neutral types, each merit a card-level glyph, or do they belong with the type they qualify under the granularity rule?"
  - "Card 2 test economy: do `TestReadSession`, `TestLocateSessions` and the `tokencount-run` fixtures duplicate behavior `tools/tokencount` tests or the existing `claudeengine` tests (`TestSessionUsage_SumsParentAndForks`) already cover? Name the covering test if so."
  - "Card 4 and card 5 test economy: do the test-local `readingEngine`, `fakeTranscripts`, `fakePricer`, `oneEngine` and the entry-builder helpers (`assistant`, `toolResult`, `bashUse`, `stamped`, `userAt`, `timedAssistant`, `userPrompt`) plan per-symbol tests, or only shared fixtures reached through public-surface tests?"
  - "Card 5 Edit glyphs: for each member whose `Intent` changes parameters, results or receiver, is the re-sign arrow present; and are `main.go` and `cost.go` listed as bare file paths while the `Intent` names specific symbols?"
  - "Card order: cards 5 and 6 both edit `CountRun` and `TestCountRun`; does card 6 depend only on earlier cards, and does any card use a symbol a later card creates?"
  - "Card 4 attack-surface: the adapters hide `Engine` behind capability values; does the `Intent` state what an adapter cannot reach and what bounds it, and does `PATTERN-shuttle-provider-seam.md` edit agree?"
  - "Fidelity to the decision record: is every Decision (D1 to D10) and Constraint carried by some card, in particular the D7 header line for an engine lacking the locator, the three wording changes (`-no-cutoff` help text, no-runs error, no-sessions error), and the `-projects` flag deletion; and does any card add scope the record does not license?"
  - "Verify section: does it run the tests of every package the cards target (`shuttleengine`, `claudeengine`, `shuttleengines`, `tools/tokencount`, `cmd/lyx` for the budget file), and is it in the plan template's order?"
---

Focus is set from a read of `00-overview.md`, the six cards and `decision-record.md`.
Entries are questions to settle, not verdicts.
