# The orch Go driver spawns the board-tidy agent after each landing

`board-tidy-after-landing` · Note · enhancement, orch, board, reed

## Seen (operator, 2026-10-23)

Once `loom-tail-review` removes PR-Align, nothing in a run names the board notes a landing superseded.
Batten's job is one task, never other board entries, so the orch spawns a Sonnet agent at each `landed` notice that reports those notes, and the operator decides (hub-wide orch rule "Board tidy after a landing").

## Idea

When the orch splits into an LLM and a Go driver (the beam's daemon), the Go driver spawns that agent after each landing, in a window of the beam's reed session, and hands its report to the orch LLM.
