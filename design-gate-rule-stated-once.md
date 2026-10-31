# The appended system prompt states the gate rule once, not twice

`gate-rule-stated-once` · Note · enhancement, shuttle, gate, claude

## Seen (hub orch, 2026-10-31)

The deny notices shuttle appends to every Claude session's system prompt carry the whole gate rule twice.
The raw go row (`raw-go` in `internal/shellguard/rows.go`) and the refuse-raw row each end in the same text, from "Gate rule: run lym gate test in the foreground" to "so the prompt cache stays warm between calls", so a session reads it twice in its system prompt.

## Idea

State the gate rule once in the appended notice, and let the second row's notice say only what it refuses.
Each row's steer, the reason shown when a call is denied, keeps the full rule, since a denied call needs the way forward at hand.
