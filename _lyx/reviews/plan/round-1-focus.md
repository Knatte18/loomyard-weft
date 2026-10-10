---
round: 1
focus:
  - "Cards 5 and 6 both edit `Run`, `Run.Wait`, `Run.pollEventsTick`, `TestRun_Wait_HeldTurnEnd` and `TestWait_LogsSessionStateBesideItsClassification`: settle whether card 6 depends on anything card 5 creates beyond `turnStartOffsets` (the card order rule), and whether the shared test rows collide."
  - "Card 3: settle whether the `Edit` glyphs for `InnerRunDeps`, `NewInnerRun` and `childWait` cover every symbol the `Intent` changes, and whether the new `Awake` seam and the suspend-subtraction state the `Intent` adds carry a stated bound on what they can skip or let through."
  - "Card 5: settle whether the hold guard, which can skip every turn end of a run on a provider that writes turn starts for load turns only, states what bounds it in the `Intent`, and whether the three card-5 test rows duplicate behaviour an existing `TestRun_Wait_HeldTurnEnd` row already covers."
  - "Fidelity: settle whether each of D1 to D5 and every Constraint in `_lyx/discussion/decision-record.md` is carried by a card, and whether card 4's and card 2's scope stays inside what the record licenses."
  - "Verify coverage: settle whether the `## verify:` commands run the tests of every package the cards target, including `internal/shedadapters`, `internal/battenshed` and `internal/shuttleengine`, and whether the section follows the plan template's order."
---

Seed focus from a read of the overview and cards 3, 5 and 6.
Cards 1, 2 and 4 were not read in depth and carry no entry beyond the fidelity and verify questions.
