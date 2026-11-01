---
round: 1
focus:
  - "D1, D3: the detached runner, the lease and the join rule add an escape from the in-process gate. Check that each Decision states what the join and the superseded runner can skip or let through (slots held, a stale result replayed) and what bounds it."
  - "D2, D4: the strand-based choice and the cap-zero path change how a call waits. Check that the bound on a forged or absent strand and on the cap fallback is stated in the Decision, and that `DefaultAwaitCap` fallback does not silently lift a guard."
  - "D5: the break-even picks the long cap from run records. Check that no fact it rests on is left for Plan or an implementer to find out, such as the record fields the median reads or the Bash ceiling the long cap is derived from, and that the evidence for each is recorded."
  - "D7, D9: agent-facing renders drop `--wait`. Check that the Constraint that every Go-run gate call carries `--wait` or an inherited slot holds against the rendering sites the Decision names, and that the rule text covers the new answer without a path that counts a 75 as a failure."
  - "Open risks, Auto-mode assumptions, Constraints: check that no line restates a Decision or is settled by one (a consistency drop), and that no Decision files a follow-up as a task or edits this task's own board entry."
  - "Completeness before leanness: for any relocation to `support-log.md`, confirm the requirement stays in `decision-record.md` first, since Plan-Write never reads the log."
---

Seed focus only; the reviewer judges the whole pair on its own.
