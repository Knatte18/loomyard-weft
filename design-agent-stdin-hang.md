# Agent Bash calls hang on stdin

`agent-stdin-hang` · Note · bug, shuttle

Agents' Bash shells hung on stdin for up to 80 minutes, holding verify gates (`board-model` #361 on 2026-10-04; `orch-cycle-hardening` and `webster-audit-recovery` on 2026-10-05).

Cause (transcript study, 2026-10-05): the model, mostly Sonnet fork subagents, emits a bare `cat > /tmp/x;` or `python3 -;` with no input attached, often as a no-op (`cat > /tmp/x 2>/dev/null; true`); 54 such calls between 2026-09-24 and 2026-10-04.
The stored commands keep their newlines and no layer rewrote them; the "flattened heredoc" first reported here was read off a display that shows newlines as spaces.
No stencil shows the shape, and lyx cannot change the habit.

`3c5bfdb04` gave every Bash call `/dev/null` as stdin through a PreToolUse(Bash) rewrite hook (`bashStdinHookCommand`, `internal/shuttleengine/claudeengine/settings.go`), so such a command ends at once while heredocs, pipes and redirects still work.

Open:

- The operator dislikes hooks: each one delays every tool call.
  Find a way to give the agent's shell `/dev/null` as stdin without a per-call hook (e.g. how shuttle launches Claude Code, or `BASH_ENV`, checking it does not cut stdin from a script fed through a pipe), and drop the hook if one works.
- If the hook stays: it extracts the command with a `sed` regex taking the last `"command":"` key in the raw payload, so a later `command` field elsewhere in the payload would be rewritten instead; parse the JSON.
