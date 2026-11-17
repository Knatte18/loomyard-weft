---
round: 1
focus:
  - "Fidelity, `_lym/discussion/decision-record.md` Constraints: for each Constraint (`PATTERN-stencil-ownership`'s edit-directive background change, `PATTERN-heddle-instructions`' `HEDDLE.md` and background changes, `PATTERN-config-layers`' new codex.yaml keys, `PATTERN-child-processes`), does some card carry it, or is it plainly outside this part's changes? Settle which Constraints no card of 1-9 touches."
  - "Fidelity, `Takes` and `Delivers` in the Goal: does any card introduce scope the decision record does not license, such as card 5's `NewWithSeams` or card 6's two exported builder entry points, and is each tied to D1 or D10?"
  - "Card 5 `Extends` on `//pattern/PATTERN-shuttle-provider-seam.md`: its `Intent` says an existing bullet gains a sentence. Does that add to the ref without changing what is there, or does it rewrite the bullet and belong under `Edit`?"
  - "Card 7 `Uses` and `Intent`: it claims `judgeCall` and `runSeedSpawn` already render through `RenderJudgePrompt` and `RenderSeedPrompt` taking the engine, so no production change is needed, and names each producer's renderer. Confirm in the code that each named renderer exists, takes the engine, and is what the launching function uses, because D1 requires an inline render to be split out otherwise."
  - "Test economy, cards 2, 6, 7: does `TestLaunchSpecLiterals_AreListed` duplicate the prompt-engine scan in `internal/shuttleengine`'s test files or another existing scan? Does `TestStandingPassages` (card 1) or `TestSpecs_MatchTheStartedSpecs` (card 6) cover behavior an existing test already covers? Name the existing test before raising a finding."
  - "Card order across 1-8: does any card use a symbol, field or fixture that a later card creates (`ListedSkills` and `Plugin`/`Hook` fields from card 2 used by card 3, `ErrHookUnrun` from card 2 used by card 4 and card 6, `newParityFixture` from card 6 used by card 7, `seatTable` from card 6 used by card 8)?"
  - "Reachability of prescribed refusals: card 2's manifest copy warnings, card 6's `PrepareEach` and `Seats` errors, card 8's `LookupSeat` refusal. Does each have a named input and a named test that reaches it, and does any named input get rejected by an upstream check?"
  - "Attack-surface bounds in `Intent`: card 5's seams, card 8's verb (it still writes the Codex home), card 2's skip of the `manifests` directory in `sweepOrphans` and `findRunByStrand`. Is what each can skip or let through stated with its bound?"
  - "`Edit` glyph hygiene: do cards 1-4 and 8 list member glyphs for each symbol their `Intent` names, and carry the re-sign arrow wherever the `Intent` changes parameters or results (`Passage` readers in card 1, `skillListing` in card 3)? Are test files edited by glyph?"
  - "Granularity: do cards 1, 6 and 7 bundle independently reviewable units, or split a type from its constructor? Check card 6, which bundles the shared seams, prompt items, comparison, seat catalog and three exported builder entry points into one card."
  - "Plan overview: do the Shared Decisions (the `seatengine` tables decision, the copy-written-by-`Runner.launch` decision) match what cards 2, 6 and 8 say, and does `## PATTERN changes` match card 9's text and the `PATTERN-engine-parity` entry the plan declares?"
---

Focus covers fidelity to the decision record, the cards' glyph and Extends typing, card order, and the refusal and attack-surface questions.
It excludes anything the plan gate checks mechanically and any missing `DependsOn` list.
