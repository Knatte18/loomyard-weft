# Shuttle reads activity and operator input from Claude Code hooks

`shuttle-hook-signals` · Task · bug, shuttle, batten, shed, reed, claude · draft

Depends on: [`usage-limit-wake`](design-usage-limit-wake.md), [`compact-inject-draft`](design-compact-inject-draft.md)

Claude Code hooks give shuttle signals it now guesses from panes and pids; this task puts them to use.
Creel already renders hooks into each Claude session's `settings.json` (`creel-hook-post`), so new entries ride the same path.

## Dead waiter in the activity reader (from B1)

Task `idle-notice-no-live-run` landed as 450bc6ff1 with diagnostics only: `ReadAgentActivity` still skips a running run whose waiter pid is dead, so the idle notice can say "no agent activity readable" while an agent works; it logs that case (Warn once per episode).
The root cause is unconfirmed.

What to do: record a `PostToolUse` activity stamp per strand and let the activity reader use it, so a working agent reads as active whatever its waiter pid says; find and fix why the waiter pid is dead, or drop the pid as the activity source.

## Operator input lost on respawn (from B7)

An instruction the operator typed into a worker's pane was lost when the tackler re-spawned that row's session: the new session started from the row's prompt alone.

What to do: a `UserPromptSubmit` hook records each prompt the operator submits to a strand, and a re-spawned row's session gets the unanswered ones carried over with its prompt.
At the least, the operator is told in a notice that the session was replaced and which instruction may be lost.

## Further hook signals

- `SubagentStart` and `SubagentStop`: a session whose fork works reads as active, not idle.
- `PreCompact` and `PostCompact`: a compaction reads as activity, and is recorded for the strand.
- Claude Code's `quota_auto_resume_*` settings: decide whether shuttle sets them, so a session resumes by itself when a usage limit resets; align with `usage-limit-wake`, which lands first.

Each signal is used only where it replaces a guess the code makes today; one that has no such use is left out, with the reason in the change description.
