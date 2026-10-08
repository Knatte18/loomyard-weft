---
round: 3
focus:
  - "In `_lyx/discussion/decision-record.md`, settle what a send that returns `ErrSubmissionNotLanded` does with the text it typed, and whether D6's later-tick retry can pass D1's idle wait while that text is still in the box; check that the clear, if any, states what it can delete."
  - "In `decision-record.md` D6 and D1, settle whether D1's interrupt and 10 s idle override apply when the gate boundary is cleared by a writer `turn_start` with no later turn end, and whether the D6 and D1 acceptance criteria agree with each other."
  - "In `decision-record.md` D2, D3 and D6, settle which check bounds the deadline overshoot, whether it sits inside the submit window, and whether a read after the last Enter is guaranteed."
  - "In `decision-record.md` D7 and its burler acceptance criterion, settle how burler stops one half by guid while that half's `Wait` is still running in the same process, and where the mark's lock lives."
  - "In `decision-record.md` Constraints (PATTERN-completion-signal) and D7, settle which marker makes the `stopped` record write visible to the negative-verdict scan, and whether the Constraint's statement about attach's classification agrees with Scope and `isTerminalOutcome`."
---

Targets the round-2 gating areas; each entry names the question only and caps no severity.
