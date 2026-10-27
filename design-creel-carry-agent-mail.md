# Creel types agent mail in full into an idle pane instead of a reminder line

`creel-carry-agent-mail` · Note · enhancement, creel

## Seen (2026-10-23)

Creel carries a message in full into the receiver's pane only when its sender is a program or EXTERNAL.
Mail from an agent strand, and any message over 4 KiB, reaches the receiver only as a reminder line naming `lym creel read` (`internal/creelengine/doc.go`), so the receiver spends one extra turn reading it before it can act.
The reminder exists so that an agent's text cannot pose as an instruction in another agent's pane.

## Operator view, not a decision (2026-10-23)

With the runs' cost in focus, everything might instead be typed into the pane directly, with no reminder and read turn; to weigh later.

## To weigh

- The extra turn per agent message against the guard that keeps one agent's text from reading as an instruction in another agent's pane.
- Delivery at a working strand's tool boundary is task `creel-hook-delivery`, which hands over agent mail in full through its hook; this note keeps only the idle pane path's reminder.
- Codex's `PostToolUse` at a tool boundary is untested (`docs/research/creel-hook-delivery-2026-10-25/report.md` gives the steps), so the hook covers Claude strands only.
- Note `creel-agents` moves agent messaging from SendMessage onto creel, which makes agent mail, and so this cost, more common.
