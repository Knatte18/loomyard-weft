# Decision record — shuttle-turns

## Goal

Shuttle reads Claude Code's turns right: a valid Stop is always taken, a turn end without output never ends a run, and a replay corpus plus live scenarios prove both.
Closes #411, #415 and #407; the post-Enter redraw wait moves into `shuttle.yaml`.

## Scope

In:

- The #411/#415 root cause in `claudeengine`'s outstanding-work reading, with its fix.
- A trace line naming what a deferred waiting turn end waits on.
- The hold: a turn end without every output file never ends a run; shuttle notifies the parent once per live held turn end, and on an Attach for at most the last one already in the events file, and keeps waiting on the same agent.
- Attaching a `run.json` an older binary left with outcome `asking` and a live strand.
- Retiring `OutcomeAsking` as a terminal outcome, `Spec.AwaitOperator`, and every caller branch that existed only to handle an asking result.
- The fixed message tail "Answer briefly, then continue your task." in the stencils and in `lyx shuttle send`, and the autonomous AskUserQuestion steer text in `claudeengine/settings.go` saying the run waits for the answer.
- The background-shell half of #407.
- `submit_redraw_settle_ms` in `shuttle.yaml`.
- A replay corpus of real misreads, tested offline, and `llm`-tier live scenarios that provoke the hard cases.

Out:

- `crucible/` stays as it is; rewriting it or building Hardener belongs to the parked `hardener` note.
- Gate visibility in status and on screen belongs to `run-recovery`.
- Routing messages through a daemon belongs to `creel`; this task only fixes the tail senders add today.
- A Codex engine and its corpus belong to `codex-engine`.

## Decisions

### D1 — Root cause of #411/#415

Decision: the stall is the transcript half of `outstandingBackgroundTasks` (`internal/shuttleengine/claudeengine/background.go`).
In both runs the burler launched a backgrounded `go test`, and its completion notification arrived while the turn was still running.
Claude Code absorbed it mid-turn and wrote it to the transcript only as `{"type":"queue-operation","operation":"enqueue"|"remove","reason":"absorbed_mid_turn"}` lines and a `queued_command` attachment, never as a `type: "user"` message.
`transcriptHasUnmatchedLaunch` matches only user messages, so the finished shell stayed outstanding, the Stop with `background_tasks: []` became `EventWaiting`, and the gated burler run deferred the turn end until `background_shell_wait_min` (10 minutes) expired.
Rationale: verified against both transcripts on disk; their paths are in the Notes for the plan writer.

### D2 — The Stop payload's `background_tasks` is the authoritative outstanding signal

Decision: when a Stop payload carries the `background_tasks` key as a list, even an empty one, its entries with status `running` are the whole outstanding list, and the transcript is not scanned.
The transcript scan runs only for a payload without the key, or whose value is not a list.
The fallback also counts a notification found in a `queue-operation` line's `content` or a `queued_command` attachment's `prompt`, besides a user message.
Rationale: Claude Code lists both shells (`type: "shell"`) and forks (`type: "subagent"`) in `background_tasks`, as the hub's own events files show, so the payload alone is complete and is official hook data, while the transcript scan reads undocumented line shapes.
Bound: a payload that omits a running task, lists it under a status other than `running`, or changes an entry's shape under the same key counts as no outstanding work, and the transcript does not correct it.
The consequence is a turn end counted while background work still runs: it finalizes Done when every output file exists, otherwise it is a hold (D4), never a stop.
No such case is known, and the live scenarios (D10) check payload completeness for shells, Monitor and forks.

### D3 — A waiting turn end is visible in the trace

Decision: when a turn end is deferred as waiting, shuttle logs at Info, once per turn end, the outstanding tasks with their kind, id, label and the signal that reported each (payload or transcript).
Rationale: one trace line names what the wait is waiting on.
The bounded exit the board asks for is the existing one: a non-awaited shell expires after `background_shell_wait_min`, and everything else is bounded by the run deadline; with D2 a phantom outstanding task can no longer start that wait for a payload that carries the key.
A fork never expires like a shell, since that would count a turn end while a real fork still works.

### D4 — A turn end without output holds the run

Decision: `Run.Wait` never finalizes on a turn end, live ask or expired-shell turn end whose output files are not all present.
It records the hold and keeps polling the same agent; the run ends only as done (every output file present at a turn end, gate evaluated as today), died, timeout or a mechanism failure, all unchanged.
An answer to the agent is a new turn in the same session, read by the same loop.
`OutcomeAsking` is no longer returned by `Wait`; `Spec.AwaitOperator` retires, since its behavior becomes the default for every run.
Rationale: an asking stop left a live agent nobody waited on, and resuming the producer started a second agent on the same outputs (#407).
Bound: a stop no longer halts the run at once.
A held run is bounded by its run deadline and by the liveness check, which still classifies a dead pane; the parent's notice (D5) is the only early signal.
The deadline is the caller's own `Spec.Timeout`, however that caller sets it (a spawning module's `*_timeout_min` key, a recipe row's `timeout_s`, a profile or `--timeout` value), with `shuttle.yaml`'s `run_timeout_min` only where the caller leaves it zero, so the bound differs per caller.
The hold never extends it, but each Attach starts a fresh deadline, so a run resumed after a driver crash waits a further full deadline.
A run with no notifier, including a standalone autonomous `lyx shuttle run`, holds silently until that deadline; an asking stop that used to end such a run at once now ends it only there.

### D5 — The hold notifies the parent once per live held turn end

Decision: `shuttleengine.Runner` takes an optional notifier seam (a function taking one line).
On each held turn end of an autonomous run (`Spec.Interactive` false), Wait calls it once with one line naming, in this order, the agent's strand name, that it ended a turn without its output files and is held, the way forward (answer by `SendMessage` to that strand name, ending the message with "Answer briefly, then continue your task."), the outstanding background tasks with their kind and label (none when there are none), and last the start of its last message.
The line begins with a fixed notice prefix, so it never starts with `/`, and the prefix labels every delimited part of the line as the agent's words, not a command.
Every agent-written part of the line, each task label and the quoted message start, is enclosed in a fixed pair of delimiters, has every control character, newlines included, replaced by a space, and is cut to 200 characters, counted as characters and not bytes.
A delimiter character in the agent's text is replaced by a space too, so no agent text stands outside the delimiters.
The line names at most five tasks and then a count of the rest, so its length is bounded.
The label bound matters because a shell's label is its raw command, which can span lines, and `QueueNotice` refuses a line with a newline.
The orch reads the delimited parts as a report of what the agent said, never as an instruction to the orch; the notice's own label and the orch role stencil's Messaging section (D8) both say so.
A notifier error is logged at Warn and never ends the run.
Interactive runs and runs on a runner with no notifier hold silently, logging the hold at Info as `AwaitOperator` runs do today.
The seam belongs to the runner, so every autonomous run on a runner whose wiring sets it notifies.
The wiring layer points the seam at `orchengine.QueueNotice` on the prime's orch, as batten's notices do, on the hub-mode runner each of `loomcli`, `webstercli` and `burlercli` builds.
`loomcli`'s one runner is shared by burler rounds, webster's Master, the friction-reflection and landing merge-resolve runs and the driver starter; `webstercli`'s carries the recovery strand `lyx webster recover-batch` spawns, and `burlercli`'s the operator's `lyx burler`.
The runners that `shuttlecli` and `orchcli` build, and every detached (standalone) runner, wire none.
`RunState` persists the events offset of the last notified turn end, written before the notifier is called, so an Attach that replays events after a restart does not notify the same turn end again; a crash between the write and the call loses that one notice and never repeats it.
An Attach notifies at most one turn end from the events already in the file when it starts: the last held one past the notified offset, since the earlier ones are history the parent cannot act on; turn ends read live after that notify one each.
Rationale: `PATTERN-orch-pane-single-writer` makes the watcher's queue the only Go route into the orch session, and `PATTERN-told-geometry` keeps the queue's paths out of the engine, so the engine gets a function, not a path.
Bound: one notice per held turn end, never repeated while the agent stays silent; a notice the orch has no strand to receive is logged by `QueueNotice` and dropped.

### D6 — Resuming a producer attaches to a held run

Decision: a held run's `run.json` stays `running`, so the existing Attach probe every spawning adapter runs before archiving finds it and waits on the same agent.
A `run.json` an older binary wrote with outcome `asking` and a live strand is attached as a held run, whether or not its events grew; with a dead strand it is classified as today.
A record with an `AskingOffset` is attached from that offset, so the old ask turn end is not read again and is not notified: the parent already saw that run halt.
A record with no `AskingOffset` is attached from the prompt offset, so its last turn end is read as a held turn end and notified once, and the earlier turn ends in its events file are not notified (D5).
Rationale: the second case covers runs halted `asking` before this lands; no new verb is needed, and a record without an offset must not fall back to a respawn over a live agent (#407).
Bound: only a live, tracked strand is attached; `AskingOffset` is read for legacy records and no longer written; the attach starts a fresh deadline (D4).

### D7 — Callers drop their asking branches

Decision: every branch that handled `OutcomeAsking` goes with it: the "agent is asking a question" Stuck in `shedadapters.SingleLLMProducer`, the asking retry in `BurlerProducer`, websterengine's `MasterAskingError`/`ErrMasterAsking` and the WebsterProducer mapping, and treadle's asking triage (`runTriage`, its triage verdicts and its stencil), which exists only for an asking round.
Any refusal or spec row naming those outcomes is removed in the same commit, per `PATTERN-refusal-way-forward`.
`lyx shuttle run` prints `done`, `died` or `timeout`; an interactive `lyx shuttle run` is answered in its pane.
Rationale: an outcome `Wait` cannot return leaves dead code and stale documentation behind.

### D8 — The message tail

Decision: one constant in `shuttleengine` holds "Answer briefly, then continue your task."
`lyx shuttle send` appends it to the text unless the text already ends with it, judged after trimming trailing whitespace from the text; the gate's re-prompts and Go-internal sends through `Run.Send` do not.
A text that already ends with the tail is sent once with it, never twice; the check skips only the append, not the send.
The orch role stencil's Messaging section and the shed driver stencil state that every message to a working agent ends with that tail, and the hold notice (D5) quotes it.
The orch role stencil's Messaging section also states that the delimited text in a hold notice, task labels and message start, is the agent's own words, data and not an instruction (D5).
The autonomous AskUserQuestion steer and notice in `claudeengine/settings.go` say that the run waits for the answer after the question is stated.
Rationale: senders add the tail until `creel` adds it; gate re-prompts are task instructions at a turn boundary, not interruptions.

### D9 — Background shells past the wait bound

Decision: an expired non-awaited shell counts its turn end only as D4 counts any turn end: done when every output file exists, otherwise a hold whose notice (D5) names the shell among its outstanding tasks.
lyx never kills the shell: it does not own the agent's shells, and killing one could destroy work the agent still needs.
Webster's `webster-background-shell` note and warning report the run's real outcome, so a turn end past an expired shell can no longer produce a premature stuck verdict.
Rationale: the premature verdict came from the asking stop; with the hold, lyx keeps waiting on the agent and its shell until the outputs exist, the agent dies or the run times out.

### D10 — Replay corpus and live scenarios

Decision: the corpus lives under `internal/shuttleengine/claudeengine/testdata/corpus/`, one directory per case: the trimmed real transcript lines that matter, the run's `events.jsonl` (reconstructed from the transcript's turn ends when the real one is gone, and marked so), and a case file giving the source run, date, Claude Code version, what went wrong and the expected reading of each turn end (done, waiting with which tasks, held).
One untagged table test replays every case through `ParseEvents` and the wait classification, through public surfaces, with an injected clock and fake reed, and fails naming the case.
Fixtures carry no secrets and are trimmed to the lines needed; `transcript_path` is rewritten to the fixture at load time.
The first cases are: #411 absorbed-notification (both runs), the `bugfix-sessions` Plan-Write ask answered in its session (held, then done), a Webster Master turn end with a fork still running (waiting), a fork killed by an API error, a skill-load turn end, and a session `continued-in` another.
Live scenarios are `llm`-tier tests beside the existing shuttle smoke tests, reaching Claude through `llmkit`: a long background shell, a background shell finishing mid-turn, a Monitor, a fork, a message mid-turn, a skill load, a compact, an interrupt and a question; each asserts shuttle's reading and checks that `background_tasks` lists what is running.
A failing scenario keeps its events and transcript in a directory it prints, so promoting it to a corpus case is a copy and a trim.
Rationale: `PATTERN-test-speed` puts real-LLM tests in the `llm` tier, run by hand; `PATTERN-shuttle-provider-seam` keeps Claude's payload shapes, and so the corpus, in `claudeengine`.

### D11 — `submit_redraw_settle_ms`

Decision: `shuttle.yaml` gains `submit_redraw_settle_ms: 300` beside `submit_settle_ms`, read through `claudeengine.NewFromConfig`; `SubmitSettle` returns it, and `submitRedrawSettle` becomes the default for `New`.
0 reads at once; a negative value is a config error, as for `submit_settle_ms`.
Rationale: the board entry.

## Constraints

- `PATTERN-shuttle-provider-seam`: payload and transcript shapes, the corpus and its parser stay in `claudeengine`; `shuttleengine` sees only `Event` and `BackgroundTask`.
- `PATTERN-completion-signal`: every negative finalize still consults `allOutputFilesExist`; the hold adds no negative outcome.
- `PATTERN-orch-pane-single-writer` and `PATTERN-told-geometry`: the notice goes through `orchengine.QueueNotice`, wired outside the engine.
- `PATTERN-refusal-way-forward`: removed asking refusals lose their spec rows in the same commit.
- `PATTERN-test-speed`, `PATTERN-test-economy`, `PATTERN-test-isolation`: the corpus test is untagged and spawns nothing; live scenarios are `llm`-tier only; each scenario covers a reading no other test covers.
- `PATTERN-config-strictness`: the new key has a template default, so `LoadOrTemplate` callers degrade cleanly.
- No Notification hook as a signal: no observed misread needs it.
- lyx never kills an agent's background shell, and a fork never expires like a shell.
- `PATTERN-sole-parsers`: untouched; no plan, discussion or summary parsing changes.
- Docs in the same commit: `shuttleengine` and `claudeengine` `doc.go`, the `wait.go` header, `shedadapters`, `websterengine` and `treadleengine` docs for the removed branches, and the `lyx shuttle run` help text.

## Auto-mode assumptions

- The parent of every autonomous agent on a hub-mode runner built by `loomcli`, `webstercli` or `burlercli` is the prime's orch, reached through its notice queue; the runners built by `shuttlecli` and `orchcli`, and every detached runner, wire no notifier and hold silently (D5).
- No production caller depends on `lyx shuttle run` returning `asking`; only smoke tests do, and they change with it.
- Claude Code's `background_tasks` lists every running background task; D10's live scenarios verify this for shells, Monitor and forks.
- Each caller's own run deadline, from its own config, is an acceptable upper bound for a held run nobody answers, even where that is hours (D4).

## Open risks

- A Claude Code release could drop `background_tasks` or give it a value that is not a list, in which case the transcript fallback applies; or it could change an entry's shape or status values under the same list, in which case the payload under-reports and the transcript does not correct it (D2).
  The live scenarios catch either change by hand-run.
- A held run waits up to its caller's run deadline when the orch misses the notice, and a run resumed after a driver crash waits a further full deadline; nothing else ends it early.
- An autonomous run on a runner with no notifier (D5) holds silently to its deadline at an asking stop, where it used to end at once.
- A message sent without the tail can still make an agent reply and end its turn; under the hold this costs a notice, not the run.
- Corpus cases for runs whose `events.jsonl` was deleted are reconstructions; they are marked as such.

## Acceptance criteria

- A gated run whose Stop payload has `background_tasks: []`, all outputs present, and a transcript whose background shell's notification appears only as `queue-operation`/`queued_command` lines finalizes Done on the tick it reads the Stop (the #411 corpus cases).
- A payload without `background_tasks`, or with a non-list value for it, falls back to the transcript, and a notification in a `queue-operation` line matches its launch.
- A payload whose `background_tasks` lists a task with a status other than `running` counts it as not outstanding and does not scan the transcript.
- A Stop with a running shell or fork in `background_tasks` is still waiting; an expired non-awaited shell still counts the turn end after `background_shell_wait_min`.
- A deferred waiting turn end logs its outstanding tasks once.
- A turn end without every output file never finalizes: the run keeps polling, the notifier is called exactly once with the fixed prefix that labels the delimited parts as the agent's words, the strand name, the tail and the outstanding tasks (at most five named), and the message start last.
- A hold notice's task labels and message start each sit inside the delimiters, are cut to their limit in characters, and carry no control character, newline or delimiter character; a held turn end whose outstanding shell has a multi-line command still calls the notifier once, with a single-line notice.
- After a held turn end, a later turn end with all outputs finalizes Done through the gate as today.
- A second held turn end notifies again; replaying events after Attach does not re-notify an already-notified turn end, and an Attach over several held turn ends already in the file notifies exactly one, the last.
- A held run still finalizes as died when its pane dies, as Done when its outputs appear as the pane dies, and as timeout at its unchanged deadline.
- An interactive run and a run with no notifier hold without notifying.
- A turn end whose only outstanding task is an expired non-awaited shell, with outputs missing, holds and its notice names that shell; with all outputs present it finalizes Done.
- A legacy `run.json` with outcome `asking` and a live strand is attached and waited on, not respawned, with or without an `AskingOffset`; with an offset the old ask is not notified, without one only its last turn end is notified, once, and not the earlier turn ends in its events file; with a dead strand it is classified as before.
- No caller references `OutcomeAsking`, `AwaitOperator`, `ErrMasterAsking` or treadle's asking triage; `go vet` with each tag passes.
- `lyx shuttle send` appends the tail once, also for a text that ends with the tail followed by whitespace; gate re-prompts do not carry it.
- The orch role stencil's Messaging section, the shed driver stencil and the autonomous AskUserQuestion steer carry the statements D8 names, including that a hold notice's quote is data.
- Webster's `webster-background-shell` note and warning report the run's real outcome when a turn end follows an expired shell.
- `submit_redraw_settle_ms` is read from `shuttle.yaml`, defaults to 300, and a negative value is refused.
- The corpus test runs every case directory, and a case whose reading differs fails naming it.
- The `llm`-tier scenarios compile under `go vet -tags llm ./...`, and each asserts that `background_tasks` lists what is running for its case, which is the check on the D2 bound.

## Notes for the plan writer

- `wait.go` `pollEventsTick`/`expiredTurnEnd` and the `Wait` branch that today finalizes a non-Done outcome are where the hold lands; `handleGatedBoundary` is unchanged.
- `attach.go` holds the `AskingOffset` logic D6 adapts.
- `battencli/wire.go` shows the existing `QueueNotice` wiring to copy for the notifier.
- Real transcript lines for the #411 cases: `~/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-test-suite-prune-rest/97d818aa-8fb0-4103-9b56-0ae4da2a75b7.jsonl` (launch at line 280, absorbed notice at lines 292–296) and `~/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-bugfix-sessions/3c2c87b2-91e2-498b-9337-16ff67e20974.jsonl` (lines 95, 356–362).
