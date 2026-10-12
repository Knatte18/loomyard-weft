---
round: 1
focus:
  - "decision-record.md D4, D5, D7, D8, D11: for each release or takeover path (same-run reacquire, hand-over to batten, idle-bound release, batten backstop, force release), does the Decision state what the lock then lets through and what bounds it, and do the bounds agree across the five Decisions?"
  - "decision-record.md D9 against D6: does a reroute that cuts a review short, followed by skip_merge_review or a skipped seam, leave the recorded last-reviewed parent tip and the merge record consistent, so no merged parent commit escapes review?"
  - "decision-record.md D5 against D9 and D12: the lock-wait rules (no time limit, pause halts blocked, wait record) are stated for Merge-In and for the D9 rows; do they agree on who waits, what the row does on a dead loop, and what a reroute does to a held lock?"
  - "decision-record.md D2 and D3: does the zero-value behaviour of the new loom.yaml keys, the BLOCKING-only escalation and the budget semantics say one consistent thing about what runs when the keys are absent?"
  - "decision-record.md Scope, Constraints, Auto-mode assumptions, Open risks, Acceptance criteria: does any content only restate a Decision, or carry deliberation that belongs in support-log.md? Apply the completeness-before-leanness test before proposing any move, since Plan-Write never reads support-log.md."
  - "support-log.md: does any entry hold a requirement or constraint that decision-record.md lacks, which Plan-Write would miss?"
---

Seed focus for round 1, set from a read of both artifacts and the discussion stencil.
Entries name where to look and which question to settle; none pre-states a verdict.
Missing signatures, `file:line` citations and exhaustive call-site lists are not findings, since the writer was told not to gather them.
