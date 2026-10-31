```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row the cards add, I traced whether a real entry point can reach it.

Findings:

1. **MEDIUM — Card 9 (D6 "the verb's write ends the wait within a second"): the file-event wake is wired on only one of the two build paths.**
   - Card 9 sets `Env.Sampler.Watch` in `loomCLI.buildLoomShed` alone, which the `step` verb drives.
   - The `run` verb builds through `loomPreRun`. That path calls `wireLanding` (`internal/loomcli/arm.go:521`) and then `newShed`, and never reaches `buildLoomShed` (`internal/loomcli/sharedbootstrap.go:361-370`).
   - Under `lym loom run`, Sampler-Session therefore waits on its timer alone. `sampler-done` then takes effect only when the timer next fires, up to a minute later (backoff ceiling).
   - Card 4's Watch test row reaches the cut-short branch only through a fake channel, so nothing covers this path.
   - Pawl's Beam-Await has the same gap, which is the precedent the card copied.
   - Fix: set the watch where both paths pass, or after `wireLanding` in `loomPreRun` too, and add a wiring assertion.

2. **LOW — Card 2 (`EntryStamp`): a goto onto the row while the run is failed there raises the stamp.**
   - The card exempts only a goto right after an entry with producer `Sampler-Session` and an outcome other than Done, which is a halted Stuck.
   - A goto is admitted from failed and awaiting states too (`internal/shedengine/goto.go:263`, `resumableHalt`).
   - Suppose the row ended in an error with no Sampler-Session history entry, for example a failed record write. The entry before a goto onto `Sampler-Session` is then `Record-Write` Done.
   - That goto counts as an entry although the run never left the row. A flag `sampler-done` wrote while the run stood failed at the row is silently ignored after it.
   - `TestEntryStamp` has no failed-at-row row.
   - D6 says a goto raises the stamp only when it comes "from another row".
   - I did not confirm whether a failed step appends a history entry (`internal/shedengine/run.go:387`, `:485` persist `nextHistory`). If it appends none, this case is reachable.

No other guard was unreachable. These were reachable:
- **Card 4:** every Call branch, wake branch and halt.
- **Card 10:** each refusal.
- **Card 11:** the sampler-strand admission and its refusals.
- **Card 8:** the perch gate in `codeBudgetZero`. `CodeReviewOff` has no other reader on a sampler path.
- **Card 12:** the running-status read in both readers.

Tool calls:
- Grepped `internal/loomcli` for `wireLanding` and watch call sites, `internal/shedengine/goto.go` for the states a goto is admitted from, and `internal/shedengine/run.go` for history appends. My context had neither the `run` verb's build path nor the goto-admissible states.
- Read `resumableHalt` in `internal/shedengine/goto.go` and the `StateFailed` persist sites in `run.go`, to see whether a failed step records history at the row.
