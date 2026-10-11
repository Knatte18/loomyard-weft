---
round: 1
focus:
  - "decision-record.md D1, D3, D4: do the routing and hand-off rules agree?
    Check that no row ever returns Done, that every BudgetExempt and counted Stuck path reaches the Bouncer, and that the budget counts judged rounds only.
    Settle whether the fix-gate-exhausted counted Stuck with an empty Path, the pass-through of a complete round and the restart cases leave the Bouncer's episode and budget consistent with D10 and the acceptance criteria."
  - "decision-record.md D4 and D11, attack surface: for the empty-review skip, the Go-authored fixer report, the acceptance record and digest, and auto-accept, does each Decision state what it can now let through and what bounds it?
    Settle whether any bypass is unbounded, such as a skip on a review the parser misreads as empty, an accept after an escalation, or a stale flag value."
  - "decision-record.md D6, D7, D9, completeness before leanness: the fixer loses the rubric, fasit, PATTERN overview and focus file.
    Settle whether a requirement the fixer needs (the fix scope's write surface, the commit rule, how a dispute is evidenced once the fasit and Do-not-flag grounds are gone) is stated in decision-record.md itself, not only in support-log.md, which Plan-Write never reads."
  - "decision-record.md D10 against the acceptance criteria: do the per-perch key semantics (0 inherits, below 0 or a resolved value below 1 refused, N judged rounds, checkpoint above budget) agree between the Decision, the Constraints and the criteria?
    Settle whether the darn perch's existing budget key and the three new perches' keys are described without contradiction."
  - "Premises: D1's rationale rests on a Done ending the segment's bounce episode and Stuck hand-offs keeping it.
    D2 rests on the one-seat table mapping to Stuck hand-offs under the Burler adapter, and D8 on there being no fixer-to-reviewer traffic.
    Settle whether any of these is a false premise against the code."
  - "Leanness and relocation: decision-record.md Scope Out, Auto-mode assumptions and Open risks, and support-log.md's Interview and Question ledger.
    Settle whether any deliberation narrative or repeated statement in decision-record.md belongs in support-log.md or nowhere, after checking that it carries no requirement the plan writer needs."
---

Seed focus for round 1.
Not seeded, by the rubric's `Do not flag` list and the discussion stencil's gathering limits: a missing Notes subsection, rejected alternatives in the record, exact signatures or `file:line` citations, and call-site enumeration.
