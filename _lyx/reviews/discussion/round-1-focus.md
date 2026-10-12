---
round: 1
focus:
  - "D1, D3, D7 (attack surface): any caller, agents included, can write a hub-wide pause that freezes runs, network calls and the orch. Settle whether the bound names what stops a forgotten or agent-issued hub pause from silently freezing everything, and whether `lyx pause --status` and `paused_since` are enough."
  - "D4 parent-lock exception: a per-run entry does not hold a network operation whose caller holds a parent lock. Settle whether that exception can let a covered run's network work through, and whether its bound is stated."
  - "D8 (guard removal): the bare loom pause, `lyx webster pause`, and landingshed's `PauseRequested` seam are removed. Settle whether the stated bound covers what a loom row waiting in the parent lock's queue can now do, and whether every way forward still names a command that exists."
  - "D9 and D10 (new automatic actions): the once-per-turn continue after an API-error stall and the post-kill stop of shuttle runs. Settle whether each is bounded on what it can fire into or stop, including a run that a pause covers or that a session shared with another step."
  - "D3, D6, D7 consistency: ownership of a session between hub and slug entries, the orch watcher's interrupt, and the caller-skip rule. Settle whether the text and the acceptance criteria agree on who interrupts and who gets the resume line, with no case left unowned."
  - "Relocation under completeness-before-leanness: look in `decision-record.md` for deliberation narrative, such as D9's note about the 2026-10-12 notice or rationale text that re-argues a choice, that could move to `support-log.md`. Extract any requirement into Decisions or Constraints first."
  - "D2 and D5 readers: settle whether each reader that must ignore a covered interval has a Decision naming it, and whether the 7-day retention bound leaves a reader that miscounts after the window."
---

Round 1 seed.
The `Do not flag` items were applied: no entry asks for rejected alternatives in the record, call-site enumeration, or a "Notes for the plan writer" change.
The writer stencil bars exact signatures, `file:line` citations and dependency lists, so no entry asks for them.
