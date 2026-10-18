# The orch watcher sees the note is written instead of waiting for a reply

`orch-note-no-reply` · Note · enhancement, orch, creel, workspaces

## Seen (2026-10-18)

- At a context cycle the orch watcher posts a request to write the orch note, and waits for the orch to answer `lyx creel post --reply <id> "written <path>"` before it compacts or clears the session.
- The reply costs the orch one more tool call per cycle, and the operator questioned why it is needed.
- The watcher already reads the session's turn ends, since a soft cycle waits for the newest turn end to be quiet.

## Operator decision (2026-10-18)

- The watcher sees by itself that the note is written: the orch's turn has ended and the named note file exists and is not empty; it then cycles with no reply.
- A `defer` reply to a soft request stays, since only the orch knows that work a cycle would lose is in flight.
