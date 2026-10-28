# Shuttle reads activity and operator input from Claude Code hooks

A working agent no longer reads as "no agent activity readable", and an instruction the operator typed into a worker's pane survives a re-spawn of that row's session.
Shuttle took both from guesses (a waiter pid, the row's prompt alone) that failed.
Claude Code hooks now supply the signals.

## What changes

- **Activity stamp.**
  Every Claude run's settings carry a `PostToolUse`, a `PreCompact` and a `PostCompact` hook that each touch one file in the run directory.
  The file's modification time is the stamp and no payload is written, so the events file does not grow.
  `PostToolUse` fires for forks too, so a working fork moves the stamp, and a compaction counts as activity.
  In a hub the creel mail hook stays a second command beside the stamp on `PostToolUse`.
- **Activity reader.**
  `readRunActivity` takes the newest of the transcript reading, the events-file reading and the stamp as `LastActivity`.
  `ReadAgentActivity` reads every run whose record reads running, whatever its waiter pid says.
  A run whose waiter is gone still logs its waiterless episode, and its reading carries `LastActivity` only; its API-error and usage-limit parts read zero, so no revive or usage-limit notice is aimed at a session no process waits on.
- **Operator input.**
  Shuttle appends the text of every verified send into a run to a sent-texts file in the run directory, before typing it.
  `claudeengine` now passes a `UserPromptSubmit` prompt's text on as an instruction, left empty for a `<task-notification>` line and for a slash command.
  A run's operator instructions are its turn-start instructions past the prompt offset, minus the first (the run's prompt), minus every text in the sent-texts file.
- **Carry-over on re-spawn.**
  A run record keeps its role key (the spec's name override, else its role) and the instructions it carried over.
  At start, before the orphan sweep, shuttle finds the newest run of the same role key that reads stopped, died or timeout, or running with its waiter gone.
  That run's carried and operator instructions, without repeats, are appended to the new prompt as one block and recorded in the new run's record, so a second re-spawn carries them again.
  The runner's notifier, when set, gets one line naming the new strand and the instructions; the line is also logged at Warn.
- Docs: `shuttleengine/doc.go`, `claudeengine/doc.go` and `battenshed/doc.go` describe the above.

## Design decisions

- The stamp is a file time, not an events-file line: a hook that writes nothing adds no growth and no parse cost.
- The waiter pid is dropped as an activity source rather than debugged.
  The root cause of the dead waiter is unconfirmed, and no waiterless warning has appeared in the hub's logs since 450bc6ff1.
  A run whose agent is gone ages like any quiet run.
- Shuttle records its own sends because the hook payload cannot tell a parent's send or a gate re-prompt from an operator's typed instruction.
- Carry-over adds text to a prompt and starts nothing.
  A done predecessor, a running one with a live waiter, an empty role key and a run directory an earlier sweep removed carry nothing over.
- Hook names, the `<task-notification>` marker and the slash-command rule stay in `claudeengine`, per `PATTERN-shuttle-provider-seam`.
  The new record fields are optional, so an older record reads as no role key and no carried instructions.

## Signals left out

- `SubagentStart` and `SubagentStop`: the `PostToolUse` stamp already moves for every tool call a fork makes, and a fork held in one long call is read through its transcript's pending call.
  A start without a stop would read a crashed fork active with no bound, and `SubagentStop` also fires for the main agent's turn segments.
- Claude Code's auto-continue-at-usage-limit setting (`autoContinueAtUsageLimit` in 2.1.312): shuttle leaves it unset.
  The usage-limit wake already revives every run after the reset, and a session continuing by itself would race that revive with no hook to tell shuttle it resumed.

## Known limits

- A run directory swept by another run's start between the stop and the re-spawn loses its instructions; the sweep's age guard of twice the startup timeout keeps that window short.
- An operator instruction typed with text identical to one shuttle sent to the same run is not carried over.

## To check by hand

- Not run: a real Claude Code session.
  Confirm that `PostToolUse` fires for a fork's tool calls, and that `PreCompact` and `PostCompact` are accepted by the installed Claude Code.
- Not run: a tackler re-spawn in a live hub.
  Type an instruction into a worker's pane, stop the run, and check the new session's prompt and the notice.
