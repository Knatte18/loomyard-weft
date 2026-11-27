```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each finding's fix covers every member of its set (fix-affected-set lens).

List of findings left unchecked: none. I checked R1, R2 and R3.

No findings.

- **R1 (callers of `reportUpdateFrom` and the "landed desktop" sites):** every site is consistent with the new rule. Card 11's signature and its per-path bullet are updated. Steps 5 and 6 still call it "over that snapshot alone" or "over the locked snapshot and the apply report", which the bullet now resolves. Card 9's `ReportUpdate` parenthesis and the first rule of `updatePlaces` are updated, and a `TestUpdatePlaces` row is added. No other card names the landed desktop or `ReportUpdate`.
- **R2 (tests that skip by design):** no other card plans a skip by design. The other matches for "skip" are a verb skipping pre-run resolution (card 12) and logged placement skips (cards 5 and 7). The overview's contract-test-tier decision is updated to the plural. Card 1:28 says "card 8's contract test compares them [the `neve-qdbus` recordings]", which still holds: those comparisons stay in `TestContract_TheRealKWinAnswersAsRecorded`.
- **R3 (registration statements and the callers of `Spawn`):** the overview decision and card 12 both carry the run-marked rule. Cards 13 and 14 call `Spawn` but say nothing about registration or runproc, so nothing in them is stale.

Tool calls:
1. Grepped plan cards 00–14 for the `reportUpdateFrom`/`ReportUpdate`/landed-desktop sites, "skip", and the registration and `RunProcessEnv` statements. My context held only the cards the fixer read, not cards 1, 5, 7, 13 or 14 in full.
