---
round: 1
focus:
  - "internal/shedadapters/burlerfix.go and internal/burlerengine/fix.go: does the fix row post one notice per fresh fix call only (never on resume), read the review only through burlerengine.ParseCurrentReview, and log a failed post at Warn without failing the round (D1)?"
  - "internal/burlerengine/fix.go and prompt.go: with a notifier told, are decision-class findings excluded from fixer work, does a decision-only review run the gates once and skip the fixer on a pass with a report naming the routing, and does a row with no notifier (lyx burler run) keep every finding as fixer work (D2)?"
  - "internal/shedrecipe and internal/loomcli/wiring.go: does the notifier reach the row through shedrecipe.Env rather than a row Env field, and does it post to the prime's orch as the LOOM program (PATTERN-recipe-setting-lookup, D1)?"
  - "internal/loomcli/decision.go: does the notice open with the fixed phrase, name the record and entry heading, and inline the entry's decision and rationale, and does the phrase match the one the parent directive admits (D3, D4)?"
  - "contracts/stencils/parent/parent-directive-parent.md and contracts/stencils/orch, webster stencils: does the admission cover only the LOOM decision notice and leave every other non-parent message untrusted, and do the Master, implementer and verify-fix wording changes drop, weaken or contradict no existing rule (D4, D5, stencil wording)?"
  - "Test economy: do the new tests in burlerengine, shedadapters and loomcli extend existing tests where one covers the surface, and is each added behavior covered without static-prose assertions?"
  - "Docs and PATTERN: do the doc.go changes cover each changed package, and does the diff stay within the record's In scope (no changes to Bouncer stop rule, rubrics, creel sender rules or arbiter path)?"
---

Focus covers the five decisions D1 to D5 plus the repository rules the record lists as constraints.
