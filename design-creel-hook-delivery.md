# Creel delivers mail to a working Claude strand at its next tool boundary through a PostToolUse hook

`creel-hook-delivery` · Task · enhancement, creel, claude · done

## Goal

Creel delivers mail to a working Claude strand at its next tool boundary, as SendMessage does, instead of only when the strand is idle (issue #871).
A darn writer works in one long turn, so a parent decision reaches it today only after the turn ends, and `lym creel interrupt` would end the turn and lose a tool call in flight such as a gate run.

## Basis

`docs/research/creel-hook-delivery-2026-10-25/report.md` tested the mechanism for Claude with scripts standing in for creel: a `PostToolUse` hook delivered every message at the next tool boundary without interrupting the running call, each message was claimed once across the hook and the idle pane path (146 of 146), and a call with no mail cost about 0.6 ms and no tokens.

## Shape (operator decisions, 2026-10-25)

- Creel stays the one router.
  While a strand has unread mail, creel keeps a flag file for it; the flag goes when no unread mail is left.
- claudeengine renders a `PostToolUse` hook into every Claude strand's settings: plain `sh` that tests the strand's flag file and exits at once when it is absent, and only when it stands calls a `lym creel` verb that claims the strand's unread mail and prints it as `hookSpecificOutput.additionalContext`, one `[creel] #<id> from <sender>: <body>` line per message.
- The hook hands over the mail itself, agent mail included, so a message it delivers needs no `lym creel read` turn.
- Each message is delivered once, by whichever path claims it first: the hook during a turn, the existing pane delivery while the strand is idle.
  A turn with no tool call ends idle and is reached by the pane path.
- Interrupt stays for stopping an agent.

## Out of scope

- Codex: its `PostToolUse` is untested, so only Claude strands get the hook; Codex strands keep the pane path.
- The idle pane path's own rules (agent mail and long messages as a reminder line naming `lym creel read`) are unchanged; note `creel-carry-agent-mail` keeps that question.
- Moving agent messaging from SendMessage onto creel (note `creel-agents`).

