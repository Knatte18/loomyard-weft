---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md (Goal, Scope, Decisions)
    summary: The re-scoped board entry's notice-channel section has no design
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md (Goal, Scope, Decisions)
    summary: The re-scoped board entry's window-per-batten section has no design
  - id: F3
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md D4, D6, D7, D8 and Scope
    summary: The step watchdog and the pair-create retry and rollback left this task's scope
---

The operator re-scoped the board entry `lifecycle-waits` by module while Discussion ran: it is now orch and batten only.
Read it fresh with `lyx board get '{"slug":"lifecycle-waits"}' --body`.
Its three sections are the orch compact reload, one notice channel to the orch, and a window per batten.
The step watchdog went back to the note `verify-watchdog`, and the pair-create retry and rollback (#433) moved to the task `fabric-pushes`.

### [BLOCKING] The notice-channel section has no design

**Location:** `_lyx/discussion/decision-record.md`, Goal, Scope and Decisions.
**Issue:** The entry's "One notice channel to the orch" section asks for: one channel per event, with batten owning state changes and the driver messaging the orch only with a question it cannot settle; an event-driven notice instead of the 30 s poll; batten without ticks, its history recording only the child's state changes with timestamps; a quiet notice judged from the agent's own activity (transcript growth, turn ends) that names an API-error turn end; and each notice carrying the time the state held and the child's history length.
No decision covers any of these; D5 only appends a wait to today's status-file quiet notice.
**Fix:** Add decisions, scope and acceptance criteria for every point of that section; recast D5 within the new quiet notice or drop it.

### [BLOCKING] The window-per-batten section has no design

**Location:** `_lyx/discussion/decision-record.md`, Goal, Scope and Decisions.
**Issue:** The entry's "A window per batten" section asks that a batten the orch starts lands in its own tmux window in the orch's session, so the window list names every run in flight; nothing in the record addresses it.
**Fix:** Add a decision, scope and acceptance criteria for it; `orch-owns-the-loop` and `runs-overview` stay out.

### [BLOCKING] The watchdog and the pair-create parts left this task's scope

**Location:** `_lyx/discussion/decision-record.md` D4, D6, D7, D8, and the Scope entries for `internal/verifyrun` and `internal/fabricengine`.
**Issue:** D4 (the wait-overrun notice, the step watchdog from #420) and D6 (Windows job-object kill) belong to the note `verify-watchdog`; D7 and D8 (Add's push retry and `rollbackAdd`'s branch deletion) belong to the task `fabric-pushes`.
**Fix:** Drop D4, D6, D7 and D8 with their scope, constraints, acceptance criteria and doc entries; keep D1–D3 on the compact reload, which fit the entry.
