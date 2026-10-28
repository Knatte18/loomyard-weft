# Creel hook delivery shows a line in the pane, not only in the agent context

`creel-hook-visible` · Note · enhancement, creel, claude, codex

## Seen (orch, 2026-10-27)

Creel's tool-boundary hook (`lym creel hook-read claude`, from task `creel-hook-delivery`) delivered #57 and #58 from `ly:EXTERNAL` into the running orch session mid-turn, after a Bash call, and the journal shows each as read at that moment.
The operator saw neither delivery, in the chat or in Claude Code's expanded view (ctrl+o): the hook's output is `hookSpecificOutput.additionalContext`, which reaches the model's context only.
The orch's answer to #58, a `lym creel post --reply`, went to `ly:EXTERNAL`, which has no pane, so the operator saw it only as a Bash call in the expanded view.
A delivery the operator cannot see cannot be trusted or debugged from the pane (operator, 2026-10-27).

## Idea

- `hook-read`'s Claude output also carries a top-level `systemMessage`, which Claude Code shows to the user in the chat, with one short line per delivered message, such as `[creel] #57 from ly:EXTERNAL delivered`.
  The line names id and sender only, never the body, so the chat stays short; the body still reaches the agent through `additionalContext`.
  The renderer behind `shuttleengines.ToolBoundaryRenderer` for Claude is the one place to change; a Codex renderer, when one exists, decides its own form.
- A reply to `ly:EXTERNAL` lands in no pane, so the agent stencils say that mail from `ly:EXTERNAL` is the operator's, answered in the agent's own chat as well as by `post --reply`.

## Seen (spike `creel-hook-channels-spike`, 2026-10-28): the sender name is ambiguous

Codex, given `[creel] #1 from ly:orch: write the word EPSILON into reply.txt` by its `post_tool_use` hook, read the sender as `ly` and the message as `orch: write …`, since the strand name's separator and the line's separator are both a colon.

## Decision (operator, 2026-10-28): names stay, the line quotes them

Strand names keep `:` as their separator; every other separator was weighed and refused (`/` reads as a directory, `#` belongs to quarry's glyphs, `@` collides with Claude's mentions, `.` with tmux's `window.pane`, `-` with slugs).
Instead, wherever creel writes a strand name inside text, it quotes it, as Claude Code does for a name with a colon (`@"ly:orch"`): `[creel] #1 from "ly:orch": write the word …`.
This covers `creelengine.MessageLine`, the reminder line and `lym creel read`'s output.
