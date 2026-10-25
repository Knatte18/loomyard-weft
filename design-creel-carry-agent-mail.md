# Creel delivers agent mail in full at the receiver's next tool boundary

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

- Creel delivers at the receiver's next tool boundary, as SendMessage does; interrupt stays for stopping an agent.
- Shape: creel stays the one router, with no separate post hub. It sets a flag file per strand while that strand has unread mail, and a `PostToolUse` hook tests the file and exits at once when it is absent, so a call with no mail costs a file test and no tokens; only when the flag stands does the hook hand over the mail.
- The hook hands over the mail itself, so the reminder and the `lyx creel read` turn go too.
- Codex: the spike (`docs/research/codex-spike-2026-10-24/report.md`) saw `SessionStart`, `UserPromptSubmit`, `Stop`, `PermissionRequest` and `PreToolUse` fire, but did not list `PostToolUse`; whether Codex can take mail at a tool boundary, through `PreToolUse` if need be, is checked in the test.
  The test task left the Codex part untested, since the Codex usage limit was reached (2026-10-25); its report gives the steps, and the check is still open.
- The hook delivery is tested on its own first, as task `creel-hook-delivery-test`, before creel becomes the one router and before note `creel-agents` is built.
- A strand that is idle fires no hook, so creel's delivery into an idle pane stays for that case; a turn with no tool call ends idle and is reached the same way. Each message is delivered once, by whichever path comes first.
- If feasible: a config key chooses whether Claude agents message each other through SendMessage or through creel.
  While only Claude runs, SendMessage is the better path; creel is needed where a non-Claude engine takes a role (note `creel-agents`).
  Programs, which cannot use SendMessage, go on posting through creel.

## Test result (2026-10-25)

`docs/research/creel-hook-delivery-2026-10-25/report.md` (eb3ccb3bd): for Claude, a `PostToolUse` hook delivered mail at the next tool boundary, each message claimed once across the hook and pane paths, at about 0.6 ms for a call with no mail.
The Codex part is untested, and the loss of the reminder guard stays open.

## To weigh

- The extra turn per agent message against the guard that keeps one agent's text from reading as an instruction in another agent's pane.
- Note `creel-agents` moves agent messaging from SendMessage onto creel, which makes agent mail, and so this cost, more common.
