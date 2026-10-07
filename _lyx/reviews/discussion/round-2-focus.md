---
round: 2
focus:
  - "Decision 3 (blocking lock for the status-side push): does the stated wait bound for a status push hold, given that `lock.AcquireWriteLock` blocks with no timeout and `internal/gitexec` sets no command timeout, and that the status seam's contract is that a push failure warns and the run continues? Check that the Decision and the Open risks entry state the same bound and that acceptance criteria cover a status push against a held lock."
  - "Decisions 1 and 2 (both-sides push entry and `commit-records`): when the code side and the records side fail with different error kinds, does the record say what the entry returns, how a caller tells which side failed, and what the `commit-records` envelope and the seam warning say? Check the envelope wording against the case where only the code side failed and the records side was pushed."
  - "Decision 5 (`lyx fabric push` and `lyx fabric sync` carrying the code side): does the record pick one both-sides entry for `push` normal mode, state what a rejection yields (error or swallowed warning), and state the bound on the prime exposure, given that `push` and `sync` push unconditionally, `Fabric.Commit` pushes only when it landed something, and `lyx config` runs `sync` automatically after an edit?"
  - "Decision 6 and Constraints: does the new PATTERN entry's own text carry the exemption for the prime's records-only per-transition push that Scope Out keeps, and do Constraints name every PATTERN entry and refusal-spec row whose message or way forward the task changes?"
---

Targets the areas where round 1 found unstated bounds, open failure dispositions and an unreconciled constraint.
