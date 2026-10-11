# Shuttle start: tolerate a stray %, show start errors, probe sooner

A ready Claude pane whose input line shows a stray `%` no longer fails a shuttle start.
A never-ready start now names its cause and the pane's last lines.
The startup probe backs off from one poll interval instead of sleeping ten, so a ready provider is seen within about a second.
A seats chair's held-turn notice is deferred for at most 10 minutes instead of 20.

Closes #549

## What changes

- **Lone `%` reads as an empty box.**
  In `internal/shuttleengine/claudeengine`, a box whose only content after the caret and side bars is the single character `%` classifies idle, and `InputBoxText` returns empty text.
  `%` beside other text, or any other single character, is still a draft.
  The `%` is most likely zsh's `PROMPT_EOL_MARK`, left on the row Claude later draws its box on; Claude never received it as input.
  The fix lives in the provider's classifier, so the pre-write guard, the startup sends and the orch watcher all get it.
  `pattern/PATTERN-pane-text-injection.md` records the narrowed guard.
- **Start errors reach the producer.**
  `shuttleengine.Result` gains `StartError`, set only on `RunGated`'s not-started branch.
  `SingleLLMProducer.mapOutcome` appends it after the existing text, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
- **Pane tail on never-ready teardown.**
  When the provider never reached ready, no cause was given and a startup capture exists, the `abandonStartup` error ends with the capture's last lines (same line count as a send failure, through the new `captureTail` in `send.go`).
  Teardowns that carry a cause, and a pane that dies during the skill-load turn, get no tail.
- **Startup probe backoff.**
  `awaitStartup` sleeps `poll_interval_ms` first, doubles while the pane capture is unchanged, caps at the dismissal gap, and resets on any capture change.
  This follows the `PATTERN-daemon-wakeups` backoff rule, which is not amended.
  A ready provider shows within the first probes because Claude's boot redraws the pane.
- **Startup-gate dismissal throttle.**
  `checkLivenessTick` replays the trust-dismiss sequence at most once per dismissal gap, `poll_interval_ms × liveness_every_n_polls` with the multiple floored to 1 as `Wait` floors it.
  Any play that sent at least one key starts the gap, failed partway or not.
  The throttle only removes key presses; the startup deadline and tick cap still end a gate that never clears.
  `template.yaml`'s comment on `liveness_every_n_polls` and `internal/shuttleengine/doc.go` say so.
- **`holdNoticeMaxDeferral`** is 10 minutes.
  Measured advisor turns ran about 1.5 to 3.5 minutes.
- **Unrelated test fix.**
  `internal/reedengine/socketfile_test.go` now builds its stale socket by binding a datagram socket, so a parallel test's forked child cannot make the path answer a stream dial.

## Design decisions to know

- A lone `%` typed by an operator as a real draft, outside `operator_quiet_s`, reads as empty and a paste can merge with it.
  The session idle probe's fallback for a strand with no run has no operator-activity hold, so there it reads idle with no operator check.
- Dismissals are spaced rather than probes, so the stray-keypress protection of the old 10-second cadence is kept at the faster probe cadence.
- `Wait`'s own probe cadence is unchanged.

## To check by hand

- The `%` as zsh's end-of-line mark is an inference: no startup capture of the failing run survived.
  If it is in fact input in Claude's box, a send pastes behind it and the prompt reaches the agent with a leading `%`.
- Another shell's end-of-line mark (`#` for a root zsh, or a custom `PROMPT_EOL_MARK`) still fails the same way.
- A gate needing more than one dismissal within 10 seconds still takes 10 seconds per extra key.
- A legitimately long advisor turn over 10 minutes now reaches the parent as a stall notice.
- Run `lyx batten run <slug> --window` and confirm the window opens in about a second after the spawn instead of about 11.
  Not run live.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run by this description step.
