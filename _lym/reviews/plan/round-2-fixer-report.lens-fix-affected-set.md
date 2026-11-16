```yaml
lens: fix-affected-set
outcome: reported
```

The lens is fix-affected-set: rerun each set-premise finding's search and check that the fix covers every member.

Unchecked findings: none. I reran the searches for F1, F2, F3, F5 and F6. F4 and F7 have no set premise.

- **F1** (every state save that could take the lease while it is already held): the plan's mentions of `saveRunState` and the lease are in cards 10, 12 and 13. Card 12 sets `leaseHeld` true for the begin's save and false for every other save of `Run`, `runBatch` and the resume, so every save in those cards is covered. `RecordBeginPartition` (card 4) only changes `st` in memory and takes no lease. The other lease holders are card 21's `RecordDelivery`, card 14's `ArchiveRunRecord` and card 20's renumber lease. None of them saves through `saveRunState` while `Run` holds the lease. Covered.
- **F2** (every place a batch is recorded done or the verify passes, where `StopRecord` must clear): those places are `endAttempt`'s pass (card 10), which the resume also reaches, `recordDoneWithoutSpawn` (card 13) and the verify re-run pass (card 13). Each now clears it, and card 5's `Stop` field text was updated to name all three. Covered.
- **F3** (the callers of the clean-tree and head checks): `endAttempt` and `checkStoppedBatch` both call `checkTreeAndHead`. Covered.
- **F5** (every writer and reader of `outcome.yaml`): card 12 writes it and card 23 reads it. Cards 3, 14 and 26 only name the path or archive the file, so none of them needs the schema. Covered.
- **F6** (every place that fills `WeaverRun`): the fix adds the fill to `loomCLI.wire`. Card 18 already covers `newTestEnv`, `envkit.FullEnv` and batten's `intentionallyNil` through "Test fixtures fill the new fields". Covered.

no findings

Tool calls:
1. Grep of the plan for `saveRunState`, the lease and `SaveState`, to rerun F1's search; my context held only the cards the fixer read.
2. Grep for `StopRecord`, recorded-done text and `WeaverRun` across all cards, to rerun F2's and F6's searches beyond the cards in my context.
3. Grep of cards 11 and 14 for state saves inside `Run`, since card 14 edits `Run` and I had not read it.
4. Grep of card 4 for `RecordBeginPartition`, to check whether the begin's partition call saves state or takes the lease.
5. Grep for every `outcome.yaml` and `OutcomePath` mention, to rerun F5's search beyond cards 12 and 23.
6. Grep of card 26 for `outcome`, the one remaining mention of `outcome.yaml` I had not read.
