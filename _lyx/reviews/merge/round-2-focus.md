---
round: 2
focus:
  - "In internal/alignment (records.go WriteAttempt and readAttemptFile, session.go Close and MarkApplied), settle whether every read-modify-write of attempt.json goes through internal/state so a rewrite keeps keys the Attempt type does not declare, as PATTERN-tolerant-records requires, and whether a test round-trips an unknown top-level key."
  - "Across the other Go-written JSON records the parent added in this merge, settle whether any is rewritten outside internal/state or a type that keeps unknown keys itself."
---

The focus targets the seam where the task's tolerant-records rule meets the parent's new alignment package.
