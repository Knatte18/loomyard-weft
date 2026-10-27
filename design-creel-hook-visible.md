# Creel hook delivery shows a line in the pane, not only in the agent context

`creel-hook-visible` · Note · enhancement, creel, claude

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
