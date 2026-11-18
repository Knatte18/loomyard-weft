```yaml
lens: way-forward
outcome: reported
```

Way-forward lens review of the plan's refusal, warning and recovery ends.

Per refusal: the warning it raises, the next step it names, and whether that step works:
- **Card 3, failed `ReplaceStrand` of the loom status strand.** It names `lym beam start`, then `lym loom start`. This works. If the old strand is gone, the retried beam start takes the add branch for the runs strand. `lym loom start` then sees the runs strand and adds the checklist beside it. If the old strand survived, the retry runs the replace-and-re-add again.
- **Card 3, failed `AddStrandUnless` in `addChecklistBesideRuns`.** It names `lym loom start`. This works for the checklist. The runs strand is in place and no status strand exists, so `ensureStatusStrandOn` adds one beside the runs strand.
- **Card 3, failed status sidecar write after the re-add.** It names `lym loom start`. This works. The sidecar does not match the strand, so `resolveStatusStrandAction` replaces the strand with the beside-runs spec once the blocking cause is fixed.
- **Card 3, failed runs sidecar write.** It names `lym beam start`, as today. This works. The runs strand is found but not current and is replaced. The checklist re-added in the same call exists, so it is not touched.
- **Card 2, add error.** It is returned as today and is unchanged. The skipped add and the failed replace are not refusals; they log or keep, as today.

Findings:

1. LOW — D1, card 3 `internal/orchcli#orchCLI.addChecklistBesideRuns` and its sidecar warning: on a blocked or failed self run, the named `lym loom start` also restarts the halted run.
   The re-add runs for every state but done, so blocked and failed pawl runs get the checklist back too, which card 3's "blocked self run" row covers.
   On a halted run neither lock is held and no live driver strand exists, so `mustSpawnDriver` is true (`internal/loomcli/bootstrap.go:196`, called at `internal/loomcli/start.go:330`). `lym loom start` then spawns a driver and resumes the stuck pawl run, which goes beyond bringing the checklist back.
   The record names `lym loom start` as this way forward (D1), so the text holds. The card's Intent could still say, for the halted case, that the step also resumes the run, so an operator who only wants the checklist knows.

2. LOW — D1, card 3 replace-failure warning: `lym loom start` is named even when the self run is done or has no seed.
   The `loomStatus != nil` replace failure returns its warning before `selfRunUnfinished` is checked. The warning names `lym loom start` in every case, including a leftover strand from a done run.
   After a done run, `lym loom start` would at best start a driver over a finished run. D1 deliberately does not bring back the checklist of a done run.
   The record's wording for this branch is unconditional ("On this branch a failed `ReplaceStrand` returns a warning naming `lym loom start` as well as `lym beam start`"), so this is a residual mismatch, not a plan deviation. A fix consistent with the record would name `lym loom start` only when the self run is unfinished; that needs a record entry, since it narrows D1's wording.

Tool calls beyond my context:
- `grep StateDone` in `internal/loomcli/start.go`: to learn whether `lym loom start` refuses a done run. My context lacked start.go's body.
- `grep StateDone` across `internal/loomcli` and for the `mustSpawnDriver(` call site in `start.go`: to confirm a halted run's start spawns a driver. My context held the predicate but not where it is called.
