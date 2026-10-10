---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md (Scope Out, D7 Bound)
    summary: The board's first #481 expectation is left out when the operator has a draft, since the reload is held and queuing is ruled out
---

### [BLOCKING] The resume steps can still run after the operator's next message

**Location:** `_lyx/discussion/decision-record.md`, Scope "Out" (the bullet "Queuing the resume steps into Claude Code's input ahead of the operator's next message") and D7's Bound ("A draft still holds, so a reload after a compaction waits while the operator is typing").

**Issue:** The board entry's Part 2 (#481) expects: "The resume steps follow the compaction boundary at once, before the operator's next message is taken, or are queued so they run ahead of it."
The design meets the first branch only when the input box is blank at the boundary.
When the operator is typing as the compaction finishes, D7 holds every reload step on the draft, the operator submits, and their message is taken by a session with no role, skills or color reloaded, which is the very failure #481 reports and the operator has complained about repeatedly.
The second branch, queuing ahead of the message, is ruled Out, so neither branch of the expectation holds in that case.

**Fix:** Cover the draft case so the resume reaches the session before the operator's next message is taken, or state in the record how the board's expectation holds there.
One possible route, for the writer to judge: deliver the resume prompt (read the role and the orch note) through Claude Code's `SessionStart` hook with the `compact` matcher, whose output enters the context right after the compaction and ahead of any next message, so only `/color` and `/reload-plugins` remain on the typed, readiness-gated path.
Whatever route is chosen, move the matching bullet out of Scope "Out" and add an acceptance criterion for the draft case.
