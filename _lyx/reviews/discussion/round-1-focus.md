---
round: 1
focus:
  - "D6 + Open risks (decision-record.md): the exemption is 'every card's every type label is Prosa'. contracts/specs/loom-plan-spec.md defines Prosa as 'file(s), no symbol target' and only reserves it for documentation by prose (line 303). Settle whether anything mechanical stops a Prosa card from targeting a non-documentation file such as a .go file, and whether the record states what an exempt generation can therefore let through unreviewed and what bounds it. Open risks names only the stencil case."
  - "Apply D12's own attack-surface question to this task's new seams: the Plan-Bouncer skip seam (D7), the deletion of planparser.CheckAppendOnly (D5), and the exempt class written into record.json (D6). For each, settle whether decision-record.md says what it can skip or let through and what bounds it, including who can write each input the skip seam reads (record.json, the live overview's first_card) and whether anything can change the cards between classification and the Plan-Bouncer skip."
  - "D7 vs D9 vs D10: the skip seam requires the highest round's first_card to equal the live overview's first_card, but D9 (Plan-Review answer key adds the highest round's findings.md) and D10 (Webster-Review range starts at the highest round's head_sha) trigger only on 'a committed round exists'. Settle whether D9 and D10 behave correctly when the live plan was not produced by that round, e.g. an operator goto Plan-Write after a rework round writes a fresh generation-0-style plan (first_card absent, Plan-Write's own archive-*/ rotation)."
  - "D14 + D2 + D6 crash-resume: D2 lists record.json among the round directory's files, D6 writes the class and first_card into record.json after the session and gate pass, and the plan-writer note suggests writing record.json last as the archive step's completion marker. Settle whether the record states one unambiguous marker that tells a resumed PR-Rework 'archive done, session begun' apart from 'archive interrupted', which D14's 'never moves a file the session wrote' requires, and whether the record.json timing across D2, D6 and D14 is consistent."
  - "Relocation pass over decision-record.md: check the Decisions' Rationale lines, Open risks and Auto-mode assumptions for deliberation narrative that belongs in support-log.md, applying completeness-before-leanness first: any requirement a moved passage carries must already sit in Decisions or Constraints. The Auto-mode assumptions list restates D3, D1, D10, D11, D12 and D13; settle whether each line adds anything the plan writer needs beyond the Decision it restates."
---

Seed rationale.

The task introduces a review bypass (the Plan-Bouncer skip seam) and removes a guard (the append-only check), so the first three entries aim at whether those are bounded and whether the three "highest committed round" consumers agree on which round governs the live generation.
The fourth targets crash convergence, where the record's own pieces place `record.json` at different moments.
The fifth gives the relocation side equal footing with gap-filling.

Checked against the rubric's Do-not-flag list: none of the entries asks for a "Notes for the plan writer" subsection, rejected alternatives in the decision record, or call-site enumeration.
Checked against the writer stencil (`loom-template-discussion.md`): none asks for exact signatures, `file:line` citations, interface shapes, dependency lists or exhaustive pattern research.
Batchifier was checked and writes no artifact (`internal/loomshed/batchifier.go`), so the new route through it adds no generation-scoped file that D1's moved set would miss.
