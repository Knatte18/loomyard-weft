# shuttleengine: sends that land, stops that are recorded

A text sent into an agent pane now waits for the session to be idle, settles before Enter, and confirms it landed.
When a send cannot land, the run retries instead of halting.
A run that lyx itself stopped is recorded as `stopped`, so a later attach probe no longer halts on it.

Closes #442
Closes #443
Closes #444

## What changes

- **Verified send** (`internal/shuttleengine`, `claudeengine`):
  - Every verified send first waits up to `send_ready_timeout_s` for an idle session: an empty input box with no running-turn hint, and, with session signals, no `turn_start` after the newest turn end.
    A session still busy at the window's end fails with the new `ErrSessionBusy`, before any key is typed.
    An unmatched `turn_start` holds a send for at most 10 s of idle pane, or until `TurnStartInterrupt` reports it interrupted.
  - Typed text gets no Enter until two box reads, `submit_settle_ms` apart, agree.
  - After Enter, the box is re-read at a doubling interval within `submit_confirm_timeout_s`, with further Enters while the box still holds the text.
    A `turn_start` past the pre-send offset confirms a submission even while the box reading is ambiguous.
  - A send that does not land fails with `ErrSubmissionNotLanded` and clears its own text from the box.
    Both errors end with the pane's last 15 non-blank lines, so the halt note shows the cause.
  - `Runner.Send` into a busy pane waits for idle like every other verified send.
- **Start** (`#442`, `#443`): a skill-load or prompt-delivery send that does not land, and a prompt-offset persist failure, tear the run down as not started.
  The record is `died`, the strand is removed, and the error wraps `ErrNotStarted` beside the send's sentinel.
  Burler's second half failing this way ends the round as that half's `NotStarted` died outcome, and the burler adapter's retry sees the agent-start transient class.
  The doc comment of `shedengine.TransientAgentStart` widens to a session that never took its first input; the class set is unchanged.
- **Gate sends** (`wait.go`): after an attach over a writer mid-turn, no gate send is typed until a turn end follows the writer's newest `turn_start`.
  A re-prompt that does not land spends no attempt and is re-sent on a later tick.
  A send inside the wait loop ends its windows by the run deadline.
- **Stop verb** (`#444`): `Run.Stop` and `Runner.StopStrand` write the record as `stopped` before removing the strand.
  A record already terminal keeps its outcome, and one reading `running` whose output files all exist keeps `running`, so attach still harvests it as finished.
  Burler's half stop, webster's leftover-strand reclaim and attach's superseded-strand removal go through it, and their own strand removers are deleted.
  An attach probe over a `stopped` record with no strand answers respawn-eligible at any directory age; an untracked strand under a young `running` record still refuses.
- **Config**: new `send_ready_timeout_s` (60) and `submit_confirm_timeout_s` (30), each a config error at 0 or negative.
  `submit_settle_ms` and `submit_redraw_settle_ms` keep their keys with reworded meaning.
- **Docs**: `doc.go` of `shuttleengine`, `claudeengine`, `burlerengine` and `websterengine`, and the new invariant `PATTERN-shuttle-stop`: outside `internal/shuttleengine`, a shuttle run's strand is stopped only through the stop verb.

## Design decisions

- `stopped` is a record-only outcome: `isTerminalOutcome` recognises it and it is never a `Result.Outcome`.
  The record is written first, so a crash between the write and the removal leaves a terminal record over a live strand, which attach already treats as respawn-eligible.
- `Run` gains a record mutex so the stop verb and the run's own `Wait` saves cannot restore `running` over `stopped`.
  A `Wait` in another process holds no mark and may overwrite `stopped` with its own terminal outcome; either answer is terminal.
- The idle check reads only the raw `turn_start` and turn-end signals, never the session-state fold, and can only delay or fail a send.
- Left alone: attach's refusal of an unknown run, orch sessions' strand removal, `lyx reed remove`, and `Runner.Inject`, which still types into a busy pane.

## To check by hand

- Not run: a real Claude Code session under machine load, the scenario of #442 and #443.
  Run a loom step with a skill-load send while the machine is busy, and confirm the send lands or the step ends as a transient not-started.
- Fixed in the last review round and not re-read by a fresh reviewer:
  - `TestStopStrand_UnrecordableStopRemovesNothing` in `internal/shuttleengine/stop_test.go` pins the guid form's refusals for a corrupt `run.json` and a read-only run directory.
    Check that its two rows reach the read and write refusals they name.
  - `awaitSettledBox` in `internal/shuttleengine/send.go` now fails with `ErrSubmissionNotLanded`, and sends no Enter, when two reads agree only at or after the submit window's close.
- `gofmt -l` lists `internal/shuttleengine/run_test.go` and `internal/shuttleengine/waiting_test.go`; neither was reformatted.
- `cmd/lyx/testdata/test-budget.yaml` is raised for `internal/shuttleengine` and other packages that gained tests.
