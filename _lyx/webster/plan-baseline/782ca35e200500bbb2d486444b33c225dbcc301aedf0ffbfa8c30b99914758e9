# Card 17 — Shell-only background work never holds a row its outputs settle

**Create:**
- `plan:internal/shuttleengine#Run.unawaitedShellsOnly` -> `func (run *Run) unawaitedShellsOnly(tasks []BackgroundTask) bool`
- `plan:internal/shuttleengine/claudeengine_test#readingWaitingHeld` -> `const readingWaitingHeld = "waiting_held"`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine#Run.expiredTurnEnd`
- `internal/shuttleengine#Config`
- `internal/shuttleengine#ShellWaitBound`
- `internal/shuttleengine#TestPollEventsTick_OutstandingBackgroundWork`
- `internal/shuttleengine#TestWait_RecordsEndedShells`
- `internal/shuttleengine#TestShellWaitMark`
- `internal/shuttleengine#TestWait_PayloadShellLogsPastWaitMinOnceAndFinishesAtTheNextTurnEnd`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Run.shellWaitActive`
- `internal/shuttleengine/claudeengine_test#checkCorpusParse`
- `internal/shuttleengine/claudeengine_test#corpusReplay.checkRead`
- `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/case.yaml`
- `internal/websterengine#masterAwaitedShellPrefix`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/websterengine/doc.go`
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
In `pollEventsTick`, a waiting turn end (`EventWaiting`) of a gated run whose outstanding list is `unawaitedShellsOnly` is judged as a plain turn end: the tick keeps the tasks as `countedTasks` and clears `waitingTasks`;
with every output file present it returns `OutcomeDone`, so the wait loop's gated Done branch evaluates the gate exactly as at a plain turn end, and a passing gate finishes the run Done with the shells in `Result.EndedShells`;
with an output file missing it returns the held turn end of that waiting turn end (its message and the offset past its line), as a plain turn end with an output missing is held.
A non-awaited shell is not the session's work product, so an output written before it is final; a fork may still write the outputs, as the lens forks do after the writer's files exist, so a fork keeps the turn waiting.

Ungated run.
D11's "finalizes as at an idle-stalled turn end" is, in this wait loop, the held turn end: a plain turn end with an output file missing reads idle-stalled and `pollEventsTick` returns it held, and the held path is what notifies the parent once per held turn end (`Run.notifyHeld`), the row's own stall handling.
`expiredTurnEnd`, for an ungated run with an output file missing whose waiting list is `unawaitedShellsOnly` and whose every shell has been outstanding at least `ShellWaitBound`, returns the held turn end of that waiting turn end (its message and the offset past its line, which `Run` keeps when `pollEventsTick` records the waiting turn end), and clears `waitingTasks`, keeping the tasks as `countedTasks`, so a later idle tick returns nothing and the held wait is not restarted; a later turn end is judged afresh.
`Config.BackgroundShellWaitMin` thereby becomes the expiry of that wait and stays the display threshold of the `background shells` mark, which now shows only on an ungated run while non-awaited shells are outstanding at staggered first-seen times, one past the bound and another not, since a lone shell expires the ungated wait at the bound it would be shown at and a gated run's shell-only waiting turn end is judged at once; its field comment, `ShellWaitBound`'s doc and the `background_shell_wait_min` comment in `template.yaml` say so.
The LLM loom driver never enters shuttle's `Wait` (`internal/loomcli/driverspec.go`) and Webster's Master names its awaited shells, so neither is cut off.

Tests.
`TestPollEventsTick_OutstandingBackgroundWork` holds the shell, fork and awaited-shell cases on the fake clock, so the D11 rows land there:
its rows "a gated run with outputs and a transcript-reported shell waits" and "…payload-reported shell waits" now expect `OutcomeDone` with the shell counted;
its rows "a transcript-reported shell keeps waiting past the bound while an output file is missing" and its payload twin now expect nothing before the bound and the held turn end, with the waiting turn end's message and offset, once the fake clock passes `ShellWaitBound`, and nothing on one further idle tick;
a new row with two shells, one outstanding past the bound and one not, expects nothing until the second passes it;
a new row for a gated run with an output file missing and a non-awaited shell expects the held turn end, with the waiting turn end's message and offset;
its rows "a fork beside a shell keeps the gated run waiting", "an awaited shell beside a shell keeps the gated run waiting", "a fork beside a shell keeps waiting past the bound" and "an awaited shell keeps waiting past the bound" stay as they are and pin that forks and awaited shells keep the current rule;
its doc comment ("keeps waiting on a shell … however long they run, and is never held", "a gated run with every output file present waits on any outstanding task") and its `//testtiming:keep` reason are reworded to what the table now pins: a fork or an awaited shell never expires at a waiting turn end, a non-awaited shell alone expires the ungated wait at the bound and leaves a gated run's waiting turn end judged as a plain turn end.
`TestWait_RecordsEndedShells`'s rows "an unawaited shell never lands in EndedShells once the next turn ends" and "a gated run with its outputs present holds the shell to the deadline and records it" now expect the run to end Done at the waiting turn end, through the gate, with the shell in `EndedShells`; the second keeps its point with an awaited shell, which the gated run still holds to the deadline and records.
`TestWait_HeldTurnEndNotice`'s row "a shell outstanding keeps waiting past the bound with no held turn end and no notice", an ungated run with an output file missing, now expects exactly one held-turn-end notice at the first tick past the bound and still ends `OutcomeTimeout` at the deadline, and is renamed to say so.
`TestWait_PayloadShellLogsPastWaitMinOnceAndFinishesAtTheNextTurnEnd` now expects one notice where it asserts none, with its message "a shell outstanding is never a held turn end" reworded to the expiry; the past-wait-min Warn line stays at one, and the run still ends Done at the agent's next turn end.
`TestShellWaitMark`'s rows "a transcript-reported shell shows from the bound", "a payload-reported shell shows from the bound" and "a later event turns it off" are rewritten over two shells outstanding at staggered first-seen times, one past the bound and one not, the one state in which the mark still shows, since a lone shell crossing the bound expires the ungated wait in that tick and clears the waiting list before `syncShellWait` reads it; its fork and awaited-shell rows stay, and its doc comment states the staggered state.
`attach_asking_test.go`'s row "running record whose waiting line was notified: no new notice once the shell bound elapsed" now reaches the expired held turn end and stays as it is: the held turn end's offset is the waiting line's, which `NotifiedOffset` already records, so no new notice is sent and the run still ends `OutcomeTimeout`.
The claudeengine corpus replay (`TestCorpus_ReplaysEachTurnEnd`) of `master-fork-running` changes at its last turn end: a gated run, its output file missing, one payload-reported shell and no awaited prefix in the replay's `Spec`, so that turn end is now judged as a plain turn end with an output missing, held at once, and the notifier is called once.
Its `reading` becomes `waiting_held` (`readingWaitingHeld`): `checkCorpusParse` reads `waiting_held` like `waiting`, an `EventWaiting` carrying the case's tasks, and `corpusReplay.checkRead` expects exactly one notification for it; the run still ends `OutcomeTimeout` at the deadline.
`claudeengine/doc.go`'s replay-corpus paragraph (its sentence "and the reading each turn end must get: done, held, or waiting with its outstanding tasks") gains `waiting_held`, a waiting turn end the wait loop judges as a plain turn end with an output missing, held at once.
The case's `what_went_wrong` sentences on the turn end staying waiting on the shell whether or not Master's output files exist, until the deadline, the liveness check or a stop, and its last turn end's `old_reading`, are reworded to D11; the four fork turn ends and the `states` list stay as they are.
`wait.go`'s file header (its "none expires into a hold or a notice", "bounded only by the run's own deadline and the liveness check" and "never an arrival" sentences) and `expiredTurnEnd`'s doc comment ("never holds it", "never an arrival") are reworded to both rules, and so is `pollEventsTick`'s doc comment ("A waiting turn end never expires: a shell of either signal and a fork keep the turn waiting however long they run", and its sentence on an ungated run done at once).
`Spec.AwaitedShellPrefixes`'s comment ("since every shell holds a gated run's turn end alike") says a prefixed shell, besides staying out of the wait marker, keeps a gated waiting turn end from being judged as a plain turn end and keeps an ungated wait from expiring.
`masterAwaitedShellPrefix`'s doc comment ("does not change when a turn end counts") says the prefix is what keeps the recovery verb's shell holding a gated turn end.
`Run.shellWaitActive`'s doc comment drops "It is display only: the wait it shows expires nothing", which is false for an ungated run whose wait expires at the same bound, and states that the mark shows only while shells are outstanding at staggered first-seen times.
The `doc.go` paragraph on waiting turn ends states both rules and the new meaning of `BackgroundShellWaitMin`, and `docs/overview.md`'s shuttle line, "a turn end with outstanding work is waiting, not held", gains the exception of a turn end whose only outstanding work is non-awaited shells.
`internal/websterengine/doc.go`'s `# Background shells in Master's wait` section is reworded to D11: the awaited prefix is what keeps the recovery verb's shell holding a gated turn end, a gated turn end waiting only on non-awaited shells is judged as a plain turn end, and `background_shell_wait_min` expires an ungated wait besides setting the log threshold.

**ImpactSummary:** two decision points inside shuttle's wait loop; Wait's signature, Result and every caller are unchanged; five existing tests in `internal/shuttleengine` are flipped or rewritten to the new rule, and one claudeengine corpus case's last turn end reads held.

**Commit:** 17: shell-only background work never holds a row its outputs settle
