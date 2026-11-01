```yaml
lens: fix-proposal
outcome: reported
```

No findings.

**F1, fixed as proposed.** I looked for a case where re-applying `RowAt` after the re-read is wrong, and found none.
- An empty `current_producer` on disk can only be a fresh start, which means the first row. `def` was resolved from that same first row, so `RowAt` gives `def.Name`, which is right.
- After a gate skip, `persistRouted` has already written a non-empty `current_producer`, so `RowAt` changes nothing.
- The halt paths at `run.go:383`, `402`, `416` and `432` use `st.CurrentProducer`. Without the fix they would have persisted an empty `current_producer`.
- `TestStep_EmptyCurrentProducerOnAGatedFirstRowKeepsTheRowWhenItHalts` should therefore fail without the fix. With `outcomeSequence(Stuck)` and no `OnStuck`, the run reaches the `case output.Halt || def.OnStuck == ""` branch. That branch persists and returns `st.CurrentProducer`, so `StepResult.Next` and the status file would both be empty. I did not run the test with the fix removed.

**F2, fixed as proposed.** `TestRowAt` exercises the empty-producers branch of `RowAt`, which has no other test. It breaks nothing elsewhere.

Further tool calls:
1. Read `internal/shedengine/run.go:318-437`. My context ended at line 320, so I could not see which field the halt paths persist.
