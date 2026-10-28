# Decision record — shuttle-hook-signals

## Goal

Shuttle reads a Claude session's work and the operator's typed instructions from Claude Code hooks instead of guessing them from a waiter pid, so a working agent never reads as "no agent activity readable" and an operator instruction survives a re-spawn of its row's session.

## Scope

- A `PostToolUse` activity stamp per run, also written by `PreCompact` and `PostCompact`, read by `shuttleengine.ReadAgentActivity`.
- `ReadAgentActivity` reads a run that reads running whatever its waiter pid says.
- A record of the texts shuttle itself types into a run, so the run's `UserPromptSubmit` lines can be split into lym's own sends and operator instructions.
- A run started for a role whose previous run of that role ended unfinished gets that run's operator instructions carried over into its prompt, and the parent is told in a notice.
- The decision on `SubagentStart`, `SubagentStop` and Claude Code's auto-continue-at-usage-limit setting.

Out of scope: a new liveness rule for any reader other than `ReadAgentActivity`, and changes to the batten notice texts.

## Decisions

### D1 — One activity stamp file per run, touched by hooks

Every Claude run's settings carry a `PostToolUse` hook, a `PreCompact` hook and a `PostCompact` hook that each touch one file in the run directory, named by a `shuttleengine` constant.
The file's modification time is the stamp; the hooks write no payload, so the events file does not grow with tool output.
Rationale: `PostToolUse` fires for every tool call of the session and of its forks, so a working session or fork moves the stamp; a compaction writes nothing else for its duration.
The `PostToolUse` creel mail hook stays a second command beside the stamp in a hub.

### D2 — The activity reader uses the stamp

`readRunActivity` takes the newest of the transcript reading, the events-file reading and the stamp's modification time as `LastActivity`.
A missing stamp reads as no stamp.

### D3 — The waiter pid stops deciding whether a run is read

`ReadAgentActivity` reads every run whose record reads running, whatever its recorded waiter pid says.
A run whose waiter is gone still logs its waiterless episode, and its reading carries `LastActivity` only: its API-error and usage-limit parts read zero, so no revive or usage-limit notice is aimed at a session no process waits on.
Bound: such a run can now only make the idle notice name an agent and its idle time instead of "no agent activity readable"; a run whose agent is gone ages like any quiet run.
Rationale: the root cause of the dead waiter is unconfirmed (no waiterless warning appears in the hub's logs since 450bc6ff1), and the stamp makes the pid unneeded as an activity source.

### D4 — Shuttle records the texts it types

Every verified send into a run with a record appends its text to a file in the run directory before typing.
Rationale: the hook payload alone cannot tell a parent's send or a gate re-prompt from an operator's typed instruction.

### D5 — An operator instruction is a typed prompt shuttle did not send

The provider-neutral turn-start signal carries the prompt's text as an instruction, which `claudeengine` leaves empty for a prompt Claude Code injects, a `<task-notification>` line, and for a slash command.
A run's operator instructions are the instructions of its turn starts past its prompt offset, minus the first one, which is the run's prompt, minus every text the run's sent-texts file holds.

### D6 — A re-spawned role inherits its predecessor's operator instructions

A run's record keeps its role key, the spec's name override or else its role, and the instructions it carried over.
At start, before the orphan sweep, shuttle finds the newest run of the same role key whose record reads stopped, died or timeout, or reads running with its waiter gone.
Its carried instructions and its operator instructions, without repeats, are appended to the new run's prompt as one block and recorded in the new run's record, so a second re-spawn carries them again.
The runner's notifier, when set, gets one line naming the new run's strand and the instructions; the line is also logged at Warn.
Bound: a done predecessor, a running one with a live waiter, a role key that is empty and a run directory an earlier sweep removed carry nothing over; the carry-over adds text to a prompt and starts nothing.

### D7 — SubagentStart and SubagentStop are left out

No hook is added for them.
Rationale: D1's `PostToolUse` stamp already moves for every tool call a fork makes, and a fork held in one long call is read through its transcript's pending call; a start without a stop would read a crashed fork active with no bound, and `SubagentStop` also fires for the main agent's turn segments.

### D8 — Shuttle does not set Claude Code's auto-continue-at-usage-limit setting

Claude Code 2.1.312 names the setting `autoContinueAtUsageLimit`; shuttle's run settings leave it unset.
Rationale: the landed usage-limit wake already wakes the orch after the reset to revive every run, and a session continuing by itself would race that revive with no hook to tell shuttle it resumed.

## Constraints

- `PATTERN-shuttle-provider-seam`: hook names, the `<task-notification>` marker and the slash-command rule stay in `internal/shuttleengine/claudeengine`.
- `PATTERN-tolerant-records`: the new run-record fields are optional, so an older record reads as no role key and no carried instructions.
- `PATTERN-test-speed`: tests stay in their package's current tier, with clocks and file times set by the test.
- `PATTERN-daemon-wakeups`: no new wait or poll is added.

## Auto-mode assumptions

- `PostToolUse` fires for a fork's tool calls in Claude Code, as the existing hook-input parsing of a subagent `PostToolUse` payload assumes.
- `PreCompact` and `PostCompact` are hook events Claude Code 2.1.312 accepts; the binary names both.
- A tackler re-spawn stops the row's previous run before the new one starts, so its record reads stopped.

## Open risks

- A run directory swept by another run's start between the stop and the re-spawn loses its instructions; the sweep's age guard of twice the startup timeout makes that window short.
- An operator instruction typed with text identical to one shuttle sent to the same run is not carried over.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - waitergone-skip-untested (class scope, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: a built run's settings carry the stamp command on `PostToolUse`, `PreCompact` and `PostCompact`, in a hub and outside one, and the creel mail hook still rides `PostToolUse` in a hub.
- D2: a run whose stamp is newer than its transcript and events reads the stamp's time as `LastActivity`.
- D3: a running run whose waiter pid is gone yields a reading with its activity and zero API-error and usage-limit parts.
- D4: a verified send into a run with a record leaves its text in the run's sent-texts file.
- D5: Claude's `UserPromptSubmit` payload yields its prompt as the instruction, and a `<task-notification>` or slash-command prompt yields none.
- D6: a start whose role's previous run reads stopped with an operator instruction past its prompt appends that instruction to the new prompt, records it in the new run, and notifies once; a done predecessor carries nothing.
- D7 and D8: the change description names both signals as left out with their reasons, and the settings carry neither a subagent hook nor the auto-continue key.
