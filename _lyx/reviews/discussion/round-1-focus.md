---
round: 1
focus:
  - "Decision 5 against Scope Out and the third auto-mode assumption: `lyx fabric push` and `sync` run in the prime too, where the code side is the operator's parent branch. Settle whether the stated bound (`lyx fabric commit` already does this) holds for verbs that push unconditionally rather than only when a commit landed, and whether it contradicts 'batten keeps the parent branch out of its automatic pushes'."
  - "Decision 3 and the Open risks lock entry: the blocking `fabric.push.lock` now gates Batten's and loom's status transitions with no timeout. Settle whether a bound is stated for a hung holder beyond 'at most one push's duration', and whether the status seam can stall a run."
  - "Decision 1 against Decision 2 and the acceptance criteria: both-sides error reporting (`ErrPushRejected` unwrapped versus other errors, records push after a code-side failure). Settle whether the three agree on what `commit-records` returns, and what the status seam warns, when both sides fail with different error kinds."
  - "Decision 4: the retry fires when the remote tip is an ancestor of or equal to local HEAD. Settle whether that condition and its bound are coherent with the acceptance criterion for a real divergence, and whether any case exists where the retry could push over remote commits."
  - "Content that belongs elsewhere: look for deliberation narrative in `decision-record.md` (Rationale lines, Auto-mode assumptions, Notes for the plan writer) that could move to `support-log.md`. Before raising a move, extract any requirement or constraint the plan writer needs into Decisions or Constraints."
---

Seeded from a read of both artifacts and the writer stencil.
Excluded per the rubric: the optional notes subsection, rejected alternatives in the record, and exact signatures, `file:line` citations or call-site lists, which the writer was told not to gather.
