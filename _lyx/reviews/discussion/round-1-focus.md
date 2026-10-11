---
round: 1
focus:
  - "decision-record.md D2 and D5: does each decision that adds a command-running edge or an automatic recovery state what it can skip or let through and what bounds it? Settle whether the D2 trust claim (a pair cannot set `deploy`) and the D5 once-per-run bound hold against the other decisions."
  - "decision-record.md D1 and D4: do the Land row's lock, marker and notice rules agree with each other and with Scope and Acceptance criteria? Settle whether any failure or restart path leaves a notice sent twice, never sent, or a guard (dirty-prime wait, prime-branch check) that the criteria do not cover."
  - "decision-record.md D6: does widening the push retry to every fabric push carry a stated bound on what it can now retry or let through, and does it stay consistent with the Constraints?"
  - "decision-record.md Decisions against Goal and Scope: is any Decision or Auto-mode assumption deliberation narrative or implementation detail the plan writer does not need, so it belongs in support-log.md instead? Before raising a relocation, extract any requirement it carries into Decisions or Constraints."
  - "decision-record.md Goal, Scope, D4 and D8: do the lists of what wakes the orch and what the orch still does agree? Settle whether D4's ready notice and D8's duties contradict the Scope's Out items."
  - "decision-record.md Open risks against Constraints and Acceptance criteria: is a risk stated that a Decision or criterion already resolves, or a Decision whose premise is an unresolved risk?"
---

Round 1 focus is the attack-surface question on the new Land row, deploy key, automatic Publish retry and widened push retry, then consistency between D1, D4, D5 and D8, then relocation candidates.
Missing rejected alternatives, a missing plan-writer notes subsection and incomplete call-site enumeration are not findings.
Exact signatures and file citations are not something the writer was told to gather, so their absence is not a finding either.
