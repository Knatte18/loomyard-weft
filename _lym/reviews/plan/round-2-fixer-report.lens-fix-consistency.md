```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checking that the change set agrees with sibling cards and cross-references.

1. MEDIUM. Card 10 adds `refs/held-records/<slug>`, but card 8 still finds the hold only through the ledger.
   - Changed text: card 10's new paragraph says `recordsPushCeiling` reads that ref "when `pushledger.HeldRecordsCommit` finds no hold".
   - The twin was left as it was. Card 8 line 16 still reads the hold only through `pushledger.HeldRecordsCommit`: "it counts the records branch only up to `<hold>^`".
   - Case: after a drop, the ledger is lost. Pushes stop short at the ref, but `RunCommitsAhead` counts the held commit as ahead. Card 5 line 54 assumes that a pair "ahead only by held commits reads level once card 8 makes `RunCommitsAhead` count no held commit", so that pair's pusher never reads level and retries for good.
   - Card 17's `Held` seam, filled with `pushledger.HeldRecordsCommit`, has the same gap, though it only decides which target is handed to `Start`.

2. LOW. Cards 20 and 21 were not given the new held-only `Dropped` case.
   - Changed text: card 4's "an entry holding only `HeldRecords` or `Dropped` is held, not owed", and card 5's and card 6's level-removal conditions, which now keep `Dropped`.
   - Card 20 line 40 and its row at line 55 still name only an entry holding `HeldRecords` as "listed ... as held".
   - Card 21 lines 40 and 55 still leave out only an entry holding `HeldRecords`.
   - A parent entry holding only `Dropped` (the landing key, or a beam pair's entry) is not covered by either.

3. LOW. Card 3 still says the summary takes its waits from the run view.
   - Changed text: card 20 now takes waits from the new `Waits` read, from `HubRunSlugs` and `ReadDriveWait`, and starts the period at "the `Waits` read's `Since`".
   - Card 3 line 38ff still says "D3's summary (card 20) lists the waiting runs from the same fields", meaning `RunView`'s `WaitSince` and the fields next to it, which `DobbyRuns` fills only for the prime's own runs.

4. LOW. The overview decision's "Applies to" is stale.
   - Changed text: the decision "a spawn starts the pusher through its command line" now says that only the landing, settle and teardown starts pass `--push-now`.
   - The teardown start is card 19's `teardownPushSeams`, and the landing's mark is set through card 12's `LandLocally`.
   - Its "Applies to" still reads "cards 6, 7 and 11" and names neither card 12 nor card 19.

Further tool calls:
- grep of the sibling cards for push-now, HeldRecords, Dropped, WaitSince and the hold reads, because my context held only the cards the fixer read.
- grep of cards 8, 13, 15, 16, 21 and 23 for `LandingCommits` and `RunCommitsAhead` reads, because my context did not hold those cards.
