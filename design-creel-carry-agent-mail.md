# Carry agent mail into the pane in full instead of a reminder and a read turn

`creel-carry-agent-mail` · Note · enhancement, creel

## Seen (2026-10-23)

Creel carries a message in full into the receiver's pane only when its sender is a program or EXTERNAL.
Mail from an agent strand, and any message over 4 KiB, reaches the receiver only as a reminder line naming `lyx creel read` (`internal/creelengine/doc.go`), so the receiver spends one extra turn reading it before it can act.
The reminder exists so that an agent's text cannot pose as an instruction in another agent's pane.

## Operator view, not a decision (2026-10-23)

With the runs' cost in focus, everything might instead be typed into the pane directly, with no reminder and read turn; to weigh later.

## To weigh

- The extra turn per agent message against the guard that keeps one agent's text from reading as an instruction in another agent's pane.
- The sender line on a carried message is what made Webster's Master refuse a decision notice (issue #849), so a carried agent message would meet the same receiver judgement.
- Note `creel-agents` moves agent messaging from SendMessage onto creel, which makes agent mail, and so this cost, more common.
