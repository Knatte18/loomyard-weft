# Shuttle reads activity and operator input from Claude Code hooks

Shuttle guessed two things it can read from Claude Code hooks instead.
`ReadAgentActivity` skipped a running run whose waiter process had died, so batten's idle notice could say "no agent activity readable" while the agent worked.
When the tackler re-spawned a row's session, an instruction the operator had typed into the old pane was lost, because the new session started from the row's prompt alone.
This change adds an activity stamp hooks write, reads running runs whatever their waiter pid says, and carries the operator's typed instructions over into a re-spawned session's prompt.

## What changes

- `claudeengine`: every run's settings carry a `PostToolUse`, a `PreCompact` and a `PostCompact` hook that run `touch <stamp>; true`.
  The stamp is the file `shuttleengine.ActivityStampFileName` names, beside the run's events file.
  A tool call, a fork's included, and a compaction now move the stamp without growing the events file.
  In a hub, creel's mail hook stays on `PostToolUse` as a second entry after the stamp.
- `shuttleengine.ReadAgentActivity`: `LastActivity` is the newest of the transcript reading, the events-file reading and the stamp's modification time.
  A run whose record reads running is read even when its recorded waiter pid is gone.
  Such a run still logs its waiterless episode, and its reading carries its activity only: its API error and usage limit read zero, so batten sends no revive or usage-limit notice to a session no process waits on.
- `shuttleengine` sends: every verified send into a run with a directory appends the text it types, as one JSON string line, to the run's `sent-texts.jsonl` before typing.
- `SessionSignal` gains `Instruction`.
  Claude's `UserPromptSubmit` sets it to the trimmed `prompt`, and leaves it empty for a `<task-notification>` prompt Claude Code submits itself and for a slash command.
- `shuttleengine` start: the run record gains `roleKey`, which is the spec's name override or else its role, and `carriedInstructions`.
  Before its orphan sweep, a start finds the newest earlier run with the same role key.
  If that run is unfinished, meaning its record reads stopped, died or timeout, or reads running with its waiter gone, the start collects its operator instructions:
  the instructions of its turn starts past the prompt offset, skipping the first turn start, which is the prompt itself, and skipping every text in its `sent-texts.jsonl`.
  It adds the instructions that run had carried itself, drops repeats, and appends the result to the new prompt under `## Operator instructions carried over`.
  The new record stores them as its own carried instructions, and the start logs one Warn line and sends the runner's notifier one line that quotes each instruction.
- `doc.go` of both packages describe the stamp, the waiterless reading, the carry-over and the two signals left out.

## Design decisions

- The stamp is a file's modification time, not a line in the events file.
  `PostToolUse` payloads carry tool output, and the events file is parsed whole on every poll.
  A stamp line with no payload would also be held back by the session-signal parser as an unpaired stamp.
- The waiter pid is no longer the activity source.
  Why the waiter dies is still unconfirmed: the hub's logs hold no waiterless Warn since 450bc6ff1 added it.
  With the stamp, the activity reading no longer needs the pid to be alive.
  The pid still decides whether a reading carries API-error and usage-limit parts.
- A predecessor is matched by role key rather than strand name.
  Reed numbers a role against the strands it holds, and the new strand's name is formed only after the prompt is written.
  Only the newest run of the key counts, and only when it is unfinished.
  A done run answered its instructions, and a running run with a live waiter is a parallel run rather than a predecessor.
- The carried instructions are kept in the new record, so a second re-spawn carries them again even after the sweep removes the first run's directory.
- `SubagentStart` and `SubagentStop` are left out.
  The `PostToolUse` stamp already moves for every tool call a fork makes, and the existing transcript reading covers a fork held in one long call.
  A start with no stop would read a crashed fork active without bound, and `SubagentStop` also fires for the main agent's own turn segments.
- `PreCompact` and `PostCompact` only touch the stamp.
  The compaction itself is already recorded for the strand by the `SessionStart` hook's `compact` source.
- Claude Code's auto-continue setting, `autoContinueAtUsageLimit` in 2.1.312, is left unset.
  The landed usage-limit wake already wakes the orch after the reset to revive each run.
  A session that continued by itself would race that revive, and no hook tells shuttle it resumed.

## To check

- The re-spawn path was not run live.
  The new tier0 tests drive `Runner.Start` against a seeded predecessor and a fake engine, covering stopped, waiterless, done, live, other-role and role-less predecessors.
- A predecessor's directory removed by another run's sweep between the stop and the re-spawn carries nothing.
  The sweep's age guard, twice the startup timeout, keeps that window short.
- An instruction the operator typed with the exact text of one shuttle sent to the same run is not carried over.
- `PreCompact` and `PostCompact` are taken as accepted hook names because the installed Claude Code 2.1.312 binary names both.
  No live compaction was observed moving the stamp.
