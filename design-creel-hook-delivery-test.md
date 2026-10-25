# Test creel mail delivery through a tool-boundary hook, for Claude and Codex

`creel-hook-delivery-test` · Task · feature, creel, claude, codex · running · PR-Review

## Goal (operator, 2026-10-25)

Test, on its own, the delivery that note `creel-carry-agent-mail` proposes, before creel becomes the one router and before note `creel-agents` is built.
Nothing goes into production: a prototype in this worktree and a report.

## What is tested

- Creel sets a flag file per strand while that strand has unread mail.
- A hook tests the flag and exits at once when it is absent; when it stands, it hands the strand its mail in full, with no reminder and no `lyx creel read` turn.
- Claude: a `PostToolUse` hook, against an agent working one long turn (several tool calls, one of them long) and against an idle agent, which creel's pane delivery still reaches.
- Codex: whether `PostToolUse` fires, else whether `PreToolUse` can carry the mail; the spike (`docs/research/codex-spike-2026-10-24/report.md`) saw `PreToolUse` fire but did not list `PostToolUse`.

## Measured

- That every message arrives, mid-turn and at idle, and is delivered exactly once when both paths race.
- Delay from post to the receiver seeing it, against Claude Code's SendMessage.
- The hook's cost per tool call with no mail (time; no tokens) and with mail.
- How the receiver treats the mail: that it reads it as a message from another strand, not as the operator, and answers by `lyx creel post --reply`.

## Output

A report in `docs/research/creel-hook-delivery-<date>/report.md` with the results and a recommendation for note `creel-carry-agent-mail`.
The prototype hook and any test strands live under the worktree's `.scratch/`; no hook is installed in the operator's or the plugins' settings.
