# Codex sessions get creel mail through hooks, as Claude sessions do

`codex-creel-hooks` · Note · feature, codex, creel, shuttle

## Seen (operator and hub orch, 2026-11-03)

Creel's hook delivery works very well for Claude sessions.
`codexengine` has none: its package doc says "no creel hooks", so every Codex strand is a pane-mail strand, and mail from another agent reaches it only as a `run lym creel read` reminder.

## What the spike found

`docs/research/codex-hook-delivery-2026-10-28/report.md` tested the channels by hand on Codex CLI 0.164.0:

- A `Stop` hook answering `decision: block` with the message as `reason` is taken at once and acted on.
- A `PostToolUse` block with the message is taken at the next step, but only when the session's instructions say `[creel]` lines are mail; without that sentence the model reads it as a failed tool call.
- An async `Stop` hook cannot wake an idle session; `codex queue --thread <id> --message …` run from outside does, at once.

## Idea

`codexengine` carries creel's delivery the way `claudeengine` does: the tool-boundary claim through a `PostToolUse` block, the turn-end delivery through a `Stop` block, the idle wake through `codex queue` from the daemon, and the `[creel]` sentence in the prompt file; the launch is then marked `CreelHooks`.

Related: project `codex-engine-trial` tests what the engine does today.
