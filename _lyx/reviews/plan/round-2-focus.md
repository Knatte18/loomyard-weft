---
round: 2
focus:
  - "Cards 5 and 6 of the plan make reads stricter: the convergence guard requires a CONVERGED round's review to parse through `burlerengine.ParseReview` (which requires `verdict:` of `APPROVED` or `BLOCKING`), and `roundUsage.Review` becomes a pointer so a record with no review half reads as missing. Settle whether every existing test fixture that writes a review or a usage record without those parts (in `internal/shedadapters`, `internal/shedrecipe`, `internal/loomrecipe`) is targeted by a card, and whether each card's ImpactSummary names the packages and fixtures the new rule rejects."
  - "Card 7 records a usage entry via `recordPass` from `runSeedSpawn` and `judgeCall`. `Bouncer.Call` also attaches to a live judge or seed through `awaitLiveJudge` and `awaitLiveSeed` and discards the `shuttleengine.Result`. Settle whether every pass whose result reached a Call gets an entry, as the decision record's usage-entry decision and the acceptance criterion require, and whether the card's targets carry any signature change with the re-sign arrow."
  - "Check that tests planned in cards 4 and 5 extend the existing table for their kind (`TestBouncerEntry_ConstructionFailures`, `TestBouncer_JudgeCall_Verdicts`) rather than duplicating a row an existing test already pins."
---

Focus targets the places where a fix to the plan adds new text; the rest of the plan needs no re-review beyond the rubric.
