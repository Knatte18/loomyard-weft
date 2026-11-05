```yaml
lens: fix-consistency
outcome: reported
```

Task: the fix-consistency lens over this round's change set.

- LOW. Card 10, Intent bullet "An ordinary re-entry while the beam waits finds the await marker standing and takes today's path", and its `TestBeamAwait` row "a re-entry with the await marker standing and the ready record standing: no re-check, the row waits". The changed third-detection bullet says a standing dobby-stop marker with no land request re-runs the re-check at entry, and it names a `refuse` whose dobby restart failed as one state that leaves that marker. The sibling bullet and the row still define an ordinary re-entry by the await marker alone and do not say "no dobby-stop marker". Read as written, the two disagree on the state where both markers and the ready record stand: one says the row waits, the other says the re-check runs. Evidence: card 10's changed bullet ("The marker also stands after a `refuse` whose dobby restart failed; the entry then re-runs the re-check…") against the unchanged bullet and row quoted above, and `internal/pawlengine/beamawait.go:347-355`, where `refuse` removes the marker only after `StartDobby` succeeds.

No other twin is stale. The card 15 Halt rows match the new step 5, the card 3 cause rule matches its new `TestReadMergeState` row, the card 12 refusal and row match, and the overview's Shared Decisions bullet names all three detections. Card 18, cards 13 and 14, and card 5's `interruptedLanding` text don't conflict with the change.

Tool calls:
- Grepped cards 01–18 (except 03, 12 and 15) for twins of the changed text (Halt, loom resume, dobby-stop, done run, caller list), because my context held only the cards the fixer read.
- Read card 18's landing section and grepped cards 05–18 for the entry-detection wording, because neither was in my context.
- Read card 5's `interruptedLanding` bullet, because card 10's new detection extends it and it was not in my context.
