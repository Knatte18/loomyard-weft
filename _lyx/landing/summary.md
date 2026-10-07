# shuttle: take every valid Stop, hold turn ends without output

Shuttle now reads Claude Code's turns right: a valid Stop is always taken, and a turn end without output never ends a run.
A replay corpus and `llm`-tier live scenarios cover both.

Closes #411
Closes #415
Closes #407

## What changes

- **Stall fix (#411, #415).**
  The transcript half of `outstandingBackgroundTasks` in `claudeengine` matched a background shell's completion notification only as a `type: "user"` message.
  Claude Code absorbs a notification that arrives mid-turn and writes it only as `queue-operation` lines and a `queued_command` attachment, so the finished shell stayed outstanding and a gated run deferred its turn end until `background_shell_wait_min`.
- **Stop payload is authoritative.**
  When a Stop payload carries `background_tasks` as a list, even an empty one, its `running` entries are the whole outstanding list and the transcript is not scanned.
  The transcript scan runs only for a payload without the key or with a non-list value, and it now also counts `queue-operation` content and `queued_command` prompts.
- **Trace line.**
  A deferred waiting turn end logs once, at Info, each outstanding task with its kind, id, label and reporting signal.
- **Hold (#407).**
  `Run.Wait` never finalizes a turn end, live ask or expired-shell turn end whose output files are not all present.
  It keeps polling the same agent; the run ends only as done, died, timeout or a mechanism failure.
  `OutcomeAsking`, `Spec.AwaitOperator` and every caller branch that handled an asking result are removed.
  That covers the `shedadapters` asking Stuck and retry, webster's `MasterAskingError`/`ErrMasterAsking`, and treadle's asking triage with its verdicts and stencil.
- **Parent notice.**
  `shuttleengine.Runner` takes an optional notifier.
  An autonomous run calls it once per live held turn end with one line: a fixed prefix, the strand name, the way forward, the outstanding tasks and the start of the last message.
  `loomcli`, `webstercli` and `burlercli` point it at `orchengine.QueueNotice` on the prime's orch; `shuttlecli`, `orchcli` and detached runners wire none and hold silently.
- **Resume.**
  A held run's `run.json` stays `running`, so the existing Attach probe finds it.
  A `run.json` an older binary left as `asking` with a live strand is attached as a held run instead of respawned.
- **Message tail.**
  `lyx shuttle send` appends "Answer briefly, then continue your task." unless the text already ends with it.
  The orch and shed driver stencils and the autonomous AskUserQuestion steer in `claudeengine/settings.go` are updated to match.
- **`shuttle.yaml`.**
  New key `submit_redraw_settle_ms` (default 300, negative is a config error), read through `claudeengine.NewFromConfig`.
- **Tests.**
  An untagged replay corpus under `claudeengine/testdata/corpus/`, run by one table test, plus `llm`-tier scenarios beside the shuttle smoke tests.
  `Clock` and `Runner.SetClock` are exported so the corpus test can inject a clock.

## Design decisions to check

- **Payload bound.**
  A payload that omits a running task, lists it under a status other than `running`, or changes an entry's shape counts as no outstanding work, and the transcript does not correct it.
  The consequence is a turn end counted early, which finalizes Done when every output file exists and otherwise holds.
  The live scenarios check payload completeness for shells, Monitor and forks.
- **Hold bound.**
  A held run is bounded by its caller's run deadline and the liveness check; each Attach starts a fresh deadline.
  A run with no notifier, including a standalone autonomous `lyx shuttle run`, holds silently to that deadline where an asking stop used to end it at once.
- **Notice hygiene.**
  Every agent-written part of the notice is delimited, stripped of control characters and delimiter characters, and cut to 200 characters; at most five tasks are named.
  The orch stencil states that the delimited text is the agent's words, not an instruction.
  The notified events offset is persisted in `RunState` before the notifier is called, so a restart never repeats a notice; an Attach notifies at most the last held turn end already in the file.
- **Legacy asking records.**
  `AskingOffset` is read for legacy records and no longer written.
  With an offset the old ask is not notified; without one only the last turn end is, once.
- **Expired shells.**
  lyx never kills an agent's background shell, and a fork never expires like a shell.
  An expired non-awaited shell counts its turn end as any other: Done with all outputs, otherwise a hold naming the shell.
- **Out of scope.**
  `crucible/` is untouched, gate visibility belongs to `run-recovery`, message routing through a daemon to `creel`, and a Codex engine to `codex-engine`.

## To check by hand

- Run the `llm`-tier scenarios (`go test -tags llm`) against a real Claude Code: long background shell, shell finishing mid-turn, Monitor, fork, mid-turn message, skill load, compact, interrupt and question.
  They were compiled under `go vet -tags llm ./...` but not run.
- Confirm that `background_tasks` lists everything running in those scenarios, which is the check on the payload bound above.
- Confirm that a hold notice reaches the prime's orch in a real `lyx loom` run.
- Corpus cases whose `events.jsonl` was gone are reconstructions from the transcript and are marked as such.
