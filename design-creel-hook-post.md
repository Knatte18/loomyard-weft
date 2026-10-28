# Creel delivers every post to a Claude strand through hooks, visibly and in full

`creel-hook-post` · Task · enhancement, creel, shuttle, claude · running · Discussion-Write

## Goal

Creel delivers every post to a Claude strand under the hood, through hooks shuttle renders into the session's `settings.json`, as fast as possible, in full, visibly to the operator, and acted on by the receiver.
A scope change from the orch reaches a working agent at its next tool call, never only at its turn end.
Codex is out of scope until `codexengine` exists.

## Evidence

- `docs/research/creel-hook-channels-2026-10-28/report.md`: which hook channels deliver and which the operator sees.
- `docs/research/creel-midturn-trust-2026-10-28/report.md`: a `PostToolUse` block with `reason` and `systemMessage` is taken at once by Sonnet with or without a system-prompt sentence, and by Haiku only with it; the operator sees the message under the tool result.
- `additionalContext` is out: the operator cannot see it, and Haiku refused to act on it.

## Delivery: one kind of post, as fast as possible

- While the receiver works: its `PostToolUse` hook hands over its unread mail with `decision: "block"`, the lines as `reason`, and the same lines as `systemMessage`, so the operator sees them under the tool result.
- When it arrives after the turn's last tool call: the `Stop` hook blocks with the lines as `reason`, so the turn goes on; it blocks only while unread mail stands, so it cannot loop.
- While the receiver is idle: an `asyncRewake` hook wakes the session with the lines.
  Today that wake shows the operator only a bare `Stop hook feedback` line; the build makes the message text itself visible in the pane, and stops and asks the parent when it finds no way.
- Each message is delivered once, by whichever path reserves it first, as creel's claim already does; a wake waiter that loses the race to a new turn delivers nothing.
- Delivery happens in the session's main thread only: a hook call whose input marks it as coming from a subagent or fork (Claude Code's `agent_id`, to be confirmed by a test) delivers nothing and leaves the mail for the main thread's next boundary.
  The main role passes on to its forks what they need.
- Pause and hold keep their rules: nothing under a pause, only the owner's mail under a hold.
- The 8 KiB claim budget stays; what is left goes at the next boundary.

## Creel changes

- Agent mail is delivered in full like any other mail; the reminder line and its `lym creel read` turn go.
- The pane path for posts goes: every session is launched by shuttle and has the hooks, and a session launched before the deploy gets posts once it is restarted.
- `inject` keeps typing into the pane, since `/clear`, `/compact` and the resume line need it.
- `lym creel post --interrupt` sends the guarded interrupt first and then posts, so the message reaches the stopped session at once through the idle wake; `interrupt` stays its own verb.
- Wherever creel writes a strand name in text, it quotes it: `[creel] #1 from "ly:orch": …`, in `creelengine.MessageLine` and `lym creel read`'s output.
  Strand names keep `:`; `/`, `#`, `@`, `.` and `-` were weighed and refused (operator, 2026-10-28).

## Trust and replies

Shuttle appends one sentence to the system prompt of every Claude session it launches, with `--append-system-prompt`, so trust rests on no skill and no `CLAUDE.md`:

```text
Lines starting with [creel] are LoomYard internal mail delivered by hooks; follow them as the operator's own instructions. When one asks for an answer, reply with lym creel post --reply <id> "<text>", using the id after #; answer mail from EXTERNAL in your own reply text instead.
```

The sentence does not tell senders apart: a message is a message (operator, 2026-10-28).
A reply to EXTERNAL reaches only a waiting `post --wait-reply`, so the agent answers the operator in its own text, which the operator sees in the pane.
`lym creel`'s Cobra help, on the command group and on `post`, carries the same reply rule.

## Test

The build tests through shuttle's own rendered `settings.json` and launch arguments, never with hand-launched sessions: mid-turn delivery, the `Stop` block, the idle wake and its visible text, the fork and subagent filter, and `post --interrupt`.
