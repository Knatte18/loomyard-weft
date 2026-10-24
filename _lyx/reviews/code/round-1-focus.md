---
round: 1
focus:
  - "internal/loomshed/patternplan.go: does the running projection start at the worktree's PATTERN.md, walk pending cards in plan order, and measure each step with pattern.Headroom against the overview before it, as the decision record's 'The running projection' says? Check done cards are never projected or flagged."
  - "internal/loomshed/patternplan.go: removal crediting. Is a removes entry applied in the first pending card whose text names its backticked ID, and an unnamed removal applied before the first card? Check the record's acceptance criteria on removal-freed bytes."
  - "internal/loomshed/patternplan.go: does the whole-plan pattern-headroom check stay unchanged in behavior, and does each pattern-card-headroom finding name its card and a way forward?"
  - "internal/planparser/patternentry.go, patternentry_test.go and plan.go: the diff touches planparser, which the record's Scope does not list. Does the answer key license each change, and does card text still go only through planparser's parsed Card and CardPatternEntries?"
  - "contracts/stencils/loom/loom-template-plan.md: this stencil change is not in the record's Scope. Check it against the board entry and record, and check that no rule the answer key did not ask to change is dropped or weakened."
  - "internal/loomshed/gates_test.go: do the table rows cover each acceptance criterion (earlier-growth-later-shrink, net-zero card, prose-named removal, unnamed removal, done card)? Check for duplicated coverage and for behavior with no row."
  - "internal/loomshed/doc.go and contracts/specs/loom-plan-spec.md: do the docs state the new check accurately and match the code, and does the commit message carry any Pattern-Departure trailer that names a real entry?"
---

Seed focus from the board entry, the decision record and the diff stat.
