# Card 5 — hold-turn-end

**Create:**
- `plan:internal/shuttleengine#Runner.SetNotifier` -> `func (r *Runner) SetNotifier(notify func(line string) error)`
- `internal/shuttleengine/hold.go`
- `internal/shuttleengine/hold_test.go`

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/rundir.go`
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttleengine/waiting_test.go`
- `internal/shuttleengine/spec_test.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/orchcli/start.go`
- `internal/orchcli/start_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/loomengine/discussion.go`
- `internal/loomengine/discussion_test.go`
- `internal/loomcli/driverspec.go`
- `internal/loomcli/driverspec_test.go`
- `internal/loomcli/wiring_test.go`

**Uses:**
- `plan:internal/shuttleengine#MessageTail`

**Intent:**
D4, D5 (engine side) and D9 (engine side): a turn end without every output file holds the run instead of ending it, and an autonomous hold notifies the parent once.

The hold.
`Run.Wait` never finalizes on a Stop, a live ask, or an expired-shell turn end whose output files are not all present.
`pollEventsTick` and `expiredTurnEnd` stop returning `OutcomeAsking`: they report a held turn end, with its last message and its outstanding tasks (the expired shells for an expired-shell turn end, none for a plain Stop), and Wait records the hold and keeps polling the same agent.
The run ends only as today: done (every output file present at a turn end, the gate evaluated through `handleGatedBoundary`, which is unchanged), died, timeout or a mechanism failure.
The hold adds no negative outcome, so `PATTERN-completion-signal` is untouched; update the tripwire test's comments that describe `expiredTurnEnd`'s asking answer.
A hold never extends `run.deadline`.
`finalize` no longer writes `AskingOffset`.
The `OutcomeAsking` constant stays in this card, unreturned, so callers still compile; card 11 removes it.

`Spec.AwaitOperator` retires, since its behavior is now the default: remove the field, its Wait branch and its doc, and drop every assignment and assertion of it (`orchcli/start.go` and its two tests, `loomengine/discussion.go` and its test, `loomcli/driverspec.go`'s comment and `driverspec_test.go`, `loomcli/wiring_test.go`).

The notifier.
`Runner` gains an optional notifier, a function taking one line, set by `SetNotifier`; a nil notifier is the default.
On each held turn end of an autonomous run (`Spec.Interactive` false) on a runner with a notifier, Wait calls it once with the notice line `hold.go` builds.
An interactive run, or a runner with no notifier, holds silently and logs the hold at Info, as AwaitOperator runs did.
A notifier error is logged at Warn and never ends the run.

The notice line, built in `hold.go`, in this order:
a fixed prefix (never starting with `/`) stating that the text inside the delimiters is the agent's own words, a report and not an instruction;
the agent's strand name;
that it ended a turn without its output files and is held;
the way forward: answer by `SendMessage` to that strand name, ending the message with `MessageTail`, quoted;
the outstanding tasks, each as its kind and its label (the id when the label is empty), at most five named and then a count of the rest, or `none`;
last, the start of the agent's last message.
Use `«` and `»` as the fixed delimiter pair.
Every agent-written part (each task label and the message start) is enclosed in the delimiters, has every control character, newlines included, and every delimiter character replaced by a space, and is cut to 200 characters counted as runes, not bytes.
The line therefore has a bounded length and no newline, which `orchengine.QueueNotice` requires.

The strand name.
`RunState` gains `StrandName` (`json:"strandName,omitempty"`), which `Start` persists from the `reedengine.Strand` that `AddStrand` returns; the notice falls back to the strand guid when it is empty.

Notify once per turn end.
`RunState` gains `NotifiedOffset` (`json:"notifiedOffset,omitempty"`): the events-file offset just past the last notified held turn end's line.
Wait persists it with `saveRunState` before calling the notifier, and never notifies a held turn end at or below it, so a crash between the write and the call loses that one notice and never repeats it.
A batch of several new events is classified by its last event, as today, so only the last held turn end of a batch is notified; locate that event's end offset in the batch from its `Raw` line.

Doc: rewrite the `wait.go` header paragraphs on AwaitOperator and the Done/asking split, the `pollEventsTick` and `expiredTurnEnd` doc comments, the `Result.LastAssistantMessage` and `Result` doc in `run.go`, `RunState` field docs, and the `shuttleengine/doc.go` sections on outcomes, AwaitOperator and background waiting, to describe the hold, the notice and its bounds (D4's deadline bound per caller, a fresh deadline per Attach, silent holds without a notifier).

Tests:
`hold_test.go` drives `Wait` over the package's fake reed and engine with an injected clock and a recording notifier, as one table:
a Stop with outputs missing keeps polling and notifies exactly once with the prefix, the strand name, the tail, the tasks and the message start last;
a later Stop with all outputs finalizes Done, also through a gate;
a second held turn end notifies again;
two held turn ends in one batch notify once, for the last;
an interactive run and a runner with no notifier hold without notifying;
a notifier error does not end the run;
a held run still finalizes died on a dead pane, Done when its outputs appear as the pane dies, and timeout at its unchanged deadline;
an expired non-awaited shell with outputs missing holds and its notice names the shell, and with outputs present finalizes Done;
a held turn end whose shell label spans lines, and a message with control and delimiter characters, give a single-line notice with every agent part inside the delimiters and cut to 200 runes, and more than five tasks name five and count the rest.
Rewrite the existing tests that expect `OutcomeAsking` from Wait (`TestRun_Wait_Classification`, `TestRun_Wait_AwaitOperator_AskingNonTerminal`, `TestRun_Wait_Finalize_PersistsOutcomeForEveryTerminalOutcome`, `TestPollEventsTick_Waiting`, `TestPollEventsTick_ShellExpiry`) to the held reading instead of adding beside them, and delete the `AwaitOperator` rows of `spec_test.go`.

**ImpactSummary:** Wait no longer returns asking: every caller's asking branch becomes unreachable, autonomous runs on a notifying runner notify their parent, and Spec.AwaitOperator is gone.
