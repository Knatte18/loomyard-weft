# Carry agent mail into the pane in full instead of a reminder and a read turn

`creel-carry-agent-mail` · Note · enhancement, creel

## Seen (2026-10-23)

Creel carries a message in full into the receiver's pane only when its sender is a program or EXTERNAL.
Mail from an agent strand, and any message over 4 KiB, reaches the receiver only as a reminder line naming `lyx creel read` (`internal/creelengine/doc.go`), so the receiver spends one extra turn reading it before it can act.
The reminder exists so that an agent's text cannot pose as an instruction in another agent's pane.

## Operator view, not a decision (2026-10-23)

With the runs' cost in focus, everything might instead be typed into the pane directly, with no reminder and read turn; to weigh later.

## Seen (2026-10-25, issue #871)

Creel delivers to a strand only when it is idle, so a receiver working one long turn gets nothing until the turn ends.
A parent decision for a darn run (`lyx loom decision add`, creel #280) stayed `posted` while the darn writer worked, though the writer works in one turn from start to finish.
Claude Code's SendMessage reached the same writer mid-turn, at its next tool boundary, without interrupting it.
`lyx creel interrupt` and `inject --interrupt` are no answer: they end the running turn and lose a tool call in flight, such as a long gate run.

## Operator decisions (2026-10-25)

- Creel delivers at the receiver's next tool boundary, as SendMessage does, for example through a `PostToolUse` hook that hands the strand its unread mail; interrupt stays for stopping an agent.
- If feasible: a config key chooses whether Claude agents message each other through SendMessage or through creel.
  While only Claude runs, SendMessage is the better path; creel is needed where a non-Claude engine takes a role (note `creel-agents`).
  Programs, which cannot use SendMessage, go on posting through creel.

## To weigh

- The extra turn per agent message against the guard that keeps one agent's text from reading as an instruction in another agent's pane.
- Note `creel-agents` moves agent messaging from SendMessage onto creel, which makes agent mail, and so this cost, more common.
