---
round: 1
focus:
  - "D1 against D7: D1's first-card test drives the D3 redundancy check and the D5 caller coverage, but D7 orders D3 and D5 after the test and its fixes. Settle whether the order is satisfiable, and where the plan writer is told those rows start passing."
  - "D4 against support-log interview item 4: the rationale says the check forced planners to invent glyphs, while the interview says the check never fires because a Create bullet is its own use. Settle whether the premise behind retiring `handle-unreferenced` is stated consistently, and whether decision-record.md alone gives the plan writer a true account."
  - "D2's bound rests on webster running cards strictly one at a time. Settle whether every path that dispatches or re-dispatches cards (rework, rebaseline, retry) shares that property, or whether the bound needs a stated limit."
  - "D5's re-sign arrow is a new grammar edge and D5's caller coverage is a new blocking gate. Settle whether each states what it can now skip or let through and what bounds it: a head that parses to the right member but misstates the signature, and a name-based scan that under- or over-reports for package-level members."
  - "D3 and D5 on the plan-gates-only rule: settle whether the decision record says where a redundancy or coverage finding is surfaced once the plan passes the gate and later state changes it (handle canonicalization at `begin-batch`, a reference set that shrinks), and whether the acceptance criteria pin that."
  - "Completeness-before-leanness: settle whether any requirement stated only in support-log.md (the card-order basis for coverage, handle-as-member handling in the pure tier, the informational severity for methods and fields) is missing from decision-record.md, since Plan-Write never reads the support log."
  - "Relocation and exclusion: settle whether Open risks, Auto-mode assumptions or Notes for the plan writer carry deliberation narrative or a decision restatement that belongs in support-log.md or nowhere."
---

Focus is a starting point for round 1, not a limit on what the reviewer may raise.
No entry names a Do-not-flag item: a missing Notes subsection, absent rejected alternatives in the decision record, and call-site enumeration are all excluded.
The stencil tells the writer not to gather exact signatures or `file:line` citations, so none of the entries asks for them.
