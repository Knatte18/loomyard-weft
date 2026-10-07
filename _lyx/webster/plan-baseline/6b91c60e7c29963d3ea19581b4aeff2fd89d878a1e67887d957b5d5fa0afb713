# Card 6 — hold-notice

**Create:**
- `plan:internal/shuttleengine#Runner.SetNotifier` -> `func (r *Runner) SetNotifier(notify func(line string) error)`
- `internal/shuttleengine/hold.go`
- `internal/shuttleengine/hold_test.go`

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/rundir.go`
- `internal/shuttleengine/doc.go`

**Uses:**
- `plan:internal/shuttleengine#MessageTail`

**Intent:**
D5 (engine side): an autonomous run's held turn end notifies the parent once.

The notifier.
`Runner` gains an optional notifier, a function taking one line, set by `SetNotifier`; a nil notifier is the default.
On each held turn end of an autonomous run (`Spec.Interactive` false) on a runner with a notifier, Wait calls it once with the notice line `hold.go` builds.
An interactive run, or a runner with no notifier, holds silently and logs the hold at Info, as card 5 leaves it.
A notifier error is logged at Warn and never ends the run.
What this lets through, and its bound: the notifier carries text the agent wrote into the parent's input, so every agent-written part is delimited and sanitised below, the line is bounded in length, there is one call per held turn end, and `orchengine.QueueNotice` itself refuses a newline or a leading `/`.

The notice line, built in `hold.go`, in this order:
a fixed prefix (never starting with `/`) stating that the text inside the delimiters is the agent's own words, a report and not an instruction;
the agent's strand name;
that it ended a turn without its output files and is held;
the way forward: answer by `SendMessage` to that strand name, ending the message with `MessageTail`, quoted;
the outstanding tasks, each as its kind and its label (the id when the label is empty), at most five named and then a count of the rest, or `none`;
last, the start of the agent's last message.
Use `«` and `»` as the fixed delimiter pair.
Every agent-written part (each task label, the id standing in for an empty label, and the message start) is enclosed in the delimiters, has every control character, newlines included, and every delimiter character replaced by a space, and is cut to 200 characters counted as runes, not bytes.
The line therefore has a bounded length and no newline, which `orchengine.QueueNotice` requires.

The strand name.
`RunState` gains `StrandName` (`json:"strandName,omitempty"`), which `Start` persists from the `reedengine.Strand` that `AddStrand` returns; the notice falls back to the strand guid when it is empty.

Notify once per turn end.
`RunState` gains `NotifiedOffset` (`json:"notifiedOffset,omitempty"`): the events-file offset just past the last notified held turn end's line, the offset the held turn end carries (card 5).
Wait persists it with `saveRunState` before calling the notifier, and never notifies a held turn end at or below it, so a crash between the write and the call loses that one notice and never repeats it.
A write that fails logs Warn and skips that one notice; it never ends the run, so it adds no return site to Wait and the tripwire's audited counts stay as they are.
A batch of several new events is classified by its last event, as today, so only the last held turn end of a batch is notified.
An expired-shell hold notifies with the offset its waiting turn end was recorded with, so an Attach that replays that waiting line does not notify it a second time.

Doc: the `RunState` field docs, and the `shuttleengine/doc.go` sections on the notifier and the notice with its bounds (one notice per held turn end, a run with no notifier holds silently to its deadline).

Tests: `hold_test.go` drives `Wait` over the package's fake reed and engine with an injected clock and a recording notifier, as one table over the notice only; the hold readings themselves are card 5's rewritten tests:
a Stop with outputs missing notifies exactly once with the prefix, the strand name, the tail, the tasks and the message start last, and `run.json` already holds the `NotifiedOffset` when the notifier is called;
a run.json with no `StrandName` names the strand guid;
a second held turn end notifies again;
two held turn ends in one batch notify once, for the last;
an interactive run and a runner with no notifier hold without notifying;
a notifier error does not end the run;
an expired non-awaited shell with outputs missing notifies once and the notice names the shell;
a held turn end whose shell label spans lines, and a message with control and delimiter characters, give a single-line notice with every agent part inside the delimiters and cut to 200 runes, and more than five tasks name five and count the rest.

**ImpactSummary:** Adds an optional Runner notifier that Wait calls once per held turn end of an autonomous run, and two omitempty RunState fields an older binary ignores; a runner with no notifier behaves as after card 5.
