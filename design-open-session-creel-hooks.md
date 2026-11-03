# A lym shuttle open session gets creel hook delivery like a run session

`open-session-creel-hooks` · Note · bug, shuttle, creel, claude

## Seen (operator, 2026-11-03)

The trial agent `ly:cs-trial-2`, opened with `lym shuttle open --role cs-trial-2`, got every message from `ly:orch` as a reminder line, `[creel] 1 unread: #611 from "ly:orch" — run lym creel read`, never the message itself.
An opened session's launch carries no creel hooks, so creel marks its strand without `CreelHooks` and treats it as a pane-mail strand (`creelengine/doc.go`, Mail).
On that path mail from an agent strand is never typed into the pane, by design, so only the reminder reaches the agent, which must run `lym creel read` itself.
Run sessions and the orch get mail through their hooks at each tool boundary and turn end.

## Expected

A session opened with `lym shuttle open` in a hub is a full lym agent: its launch carries creel's hook delivery (the tool-boundary claim, the turn-end Stop delivery and the idle waiter) and its strand is marked `CreelHooks`, so mail reaches it the way it reaches a run session.
