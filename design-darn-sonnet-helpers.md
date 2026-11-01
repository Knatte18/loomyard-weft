# Agents may use subagents and forks; only headless claude -p stays refused

`darn-sonnet-helpers` · Task · enhancement, claude, shuttle, stencil · running · Code-Burler-Fix

## Seen (orch, 2026-11-01)

`claudeengine` denied the in-process Agent tool in every run by a PreToolUse hook, or let only forks through in fork-mode runs (`steerAgentDeny`, `steerAgentNonForkDeny` and their notices in `settings.go`), on the reasoning that all work must stay visible in the run's pane.
So a darn writer at 274k context could not hand per-package work to Sonnet helpers, and a Plan writer could not hand its lookups to a fresh subagent: a fork inherits the whole context, so it saves nothing.
The orch set `deny_agent_tool: false` in the hub's `claude.yaml` on 2026-11-01 as a stopgap.

## Design (operator, 2026-11-01)

- Subagents and forks are allowed in every run: Claude Code shows them in the session's own pane, and lym reads their transcripts under the session's `subagents` directory.
- What hides work is a headless `claude -p` (or `--print`) launch, and that stays refused: a PreToolUse Bash deny refuses a command that launches `claude` headless, with a notice saying why and that subagents are the way.
- The Agent deny, its notices and the `deny_agent_tool` key go, with `Spec.AllowAgentTool` and whatever else only served them; the Webster fork guard (`steerWebsterForkDeny`) stays.
- The stencils say when to use a subagent: a fresh Sonnet subagent for lookups and for batches of independent per-package work, each owning its files, launched together in one message, with the gate rule and its timeout; the work that needs the overview stays in the session.
- Docs and the system-prompt notices describe the new rule.
