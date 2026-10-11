# Injected text merges with the operator draft

`pane-inject-race` · Note · bug, shuttle, orch

## Seen (2026-10-11, operator)

Text lyx injects into a Claude pane (batten notices, resume pointers, driver resume lines, `/clear`, `/compact`, `/reload-plugins`) regularly merges with what the operator is typing, or wipes it.
The draft guard checks the input box once, before typing, and the send then spends about a second typing into the pane (Escape, 300 ms sleep, text, settle reads, Enter) with nothing stopping the operator from typing in between.
The first Enter of a verified send (`confirmSubmitted`, `internal/shuttleengine/send.go:346`) is never checked against the box, so a merged draft is submitted.
`boxHoldsSentText` (`internal/shuttleengine/run.go`) only matches the leading 48 characters, so a merged box reads as "holds the sent text" and gets further Enters.
`ClearSession`, `CompactSession` and `ReloadPlugins` (`internal/shuttleengine/session.go`) type the command and a blind Enter on the strength of the caller's earlier probe, so `/clear` can land on an operator draft.
`judgeReadiness` (`internal/shuttleengine/readiness.go:83`) reads ready when no box is found, so a menu or picker on screen passes the guard, and the leading Escape then closes it.
The leading Escape can complete a double-Esc that clears the operator's draft, or interrupt a turn the operator just started.

## Idea

- Read the box right before every Enter and press Enter only when it holds exactly the sent text (or the paste placeholder); otherwise send no Enter and report the send as not landed.
- Route the slash-command, color and session-rename sequences through that same verified path.
- Treat an unreadable box as not idle in `judgeReadiness`.
- Drop the leading Escape from `TypeSequence`, since the box was just proven empty.
- Defer an injection while the attached client's `#{client_activity}` is under about 5 s old and the target pane is its active pane.
- Later: an atomic bracketed paste with its Enter in one tmux command list, and file-based notice delivery to the orch through a Stop hook.
