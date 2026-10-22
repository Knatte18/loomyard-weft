# Card 17 — Shell-only background work never holds a row its outputs settle

**Create:**
- `plan:internal/shuttleengine#Run.unawaitedShellsOnly` -> `func (run *Run) unawaitedShellsOnly(tasks []BackgroundTask) bool`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine#Run.expiredTurnEnd`
- `internal/shuttleengine#Config`
- `internal/shuttleengine#ShellWaitBound`
- `internal/shuttleengine#TestPollEventsTick_OutstandingBackgroundWork`
- `internal/shuttleengine#TestWait_RecordsEndedShells`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/shuttleengine#Run.recordWaiting`
- `internal/shuttleengine#Run.awaitedShell`
- `internal/shuttleengine#shellWaitFixture`

**Intent:**
D11, over the provider-neutral `BackgroundTask` kinds (`PATTERN-shuttle-provider-seam`).
`unawaitedShellsOnly` reports whether a non-empty outstanding list holds no fork and no shell `Spec.AwaitedShellPrefixes` names.
Forks and awaited shells keep the current rule, and an empty outstanding list is untouched, so `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival`, whose waiting turn end lists nothing, keeps its rows.

Gated run.
In `pollEventsTick`, a waiting turn end (`EventWaiting`) of a gated run whose outstanding list is `unawaitedShellsOnly` is an arrival: the tick keeps the tasks as `countedTasks`, clears `waitingTasks` and returns `OutcomeDone`, so the wait loop's gated Done branch evaluates the gate exactly as at a plain turn end, and a passing gate finishes the run Done with the shells in `Result.EndedShells`.
A non-awaited shell is not the session's work product, so an output written before it is final; a fork may still write the outputs, as the lens forks do after the writer's files exist, so a fork keeps the turn waiting.

Ungated run.
D11's "finalizes as at an idle-stalled turn end" is, in this wait loop, the held turn end: a plain turn end with an output file missing reads idle-stalled and `pollEventsTick` returns it held, and the held path is what notifies the parent once per held turn end (`Run.notifyHeld`), the row's own stall handling.
`expiredTurnEnd`, for an ungated run with an output file missing whose waiting list is `unawaitedShellsOnly` and whose every shell has been outstanding at least `ShellWaitBound`, returns the held turn end of that waiting turn end (its message and the offset past its line, which `Run` keeps when `pollEventsTick` records the waiting turn end), and clears `waitingTasks`, keeping the tasks as `countedTasks`, so a later idle tick returns nothing and the held wait is not restarted; a later turn end is judged afresh.
`Config.BackgroundShellWaitMin` thereby becomes the expiry of that wait and stays the display threshold of every other wait; its field comment, `ShellWaitBound`'s doc and the `background_shell_wait_min` comment in `template.yaml` say so.
The LLM loom driver never enters shuttle's `Wait` (`internal/loomcli/driverspec.go`) and Webster's Master names its awaited shells, so neither is cut off.

Tests.
`TestPollEventsTick_OutstandingBackgroundWork` holds the shell, fork and awaited-shell cases on the fake clock, so the D11 rows land there:
its rows "a gated run with outputs and a transcript-reported shell waits" and "…payload-reported shell waits" now expect `OutcomeDone` with the shell counted;
its rows "a transcript-reported shell keeps waiting past the bound while an output file is missing" and its payload twin now expect nothing before the bound and the held turn end, with the waiting turn end's message and offset, once the fake clock passes `ShellWaitBound`, and nothing on one further idle tick;
a new row with two shells, one outstanding past the bound and one not, expects nothing until the second passes it;
its rows "a fork beside a shell keeps the gated run waiting", "an awaited shell beside a shell keeps the gated run waiting", "a fork beside a shell keeps waiting past the bound" and "an awaited shell keeps waiting past the bound" stay as they are and pin that forks and awaited shells keep the current rule;
its `//testtiming:keep` reason is reworded to what the table now pins: a fork or an awaited shell never expires at a waiting turn end, a non-awaited shell alone expires the ungated wait at the bound and is an arrival for a gated run.
`TestWait_RecordsEndedShells`'s rows "an unawaited shell never lands in EndedShells once the next turn ends" and "a gated run with its outputs present holds the shell to the deadline and records it" now expect the run to end Done at the waiting turn end, through the gate, with the shell in `EndedShells`; the second keeps its point with an awaited shell, which the gated run still holds to the deadline and records.
The `doc.go` paragraph on waiting turn ends states both rules and the new meaning of `BackgroundShellWaitMin`, and `docs/overview.md`'s shuttle line, "a turn end with outstanding work is waiting, not held", gains the exception of a turn end whose only outstanding work is non-awaited shells.

**ImpactSummary:** two decision points inside shuttle's wait loop; Wait's signature, Result and every caller are unchanged; two existing tests have rows flipped to the new rule.

**Commit:** 17: shell-only background work never holds a row its outputs settle
