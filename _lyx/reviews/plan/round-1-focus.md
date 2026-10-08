---
round: 1
focus:
  - "Fidelity, D2: card 1 and card 2 carry every D2 clause, including the marker recognition (inside `## Open risks`, outside fences, right-trimmed), the result check (same H2 sequence, one well-formed pair per segment, other segments byte-identical), the five malformed-marker-set refusals, one-line rendering, and the skipped-review no-op. Is each clause carried by a card, and does any card widen scope past it?"
  - "Fidelity, D1 and D7: card 5 carries the earlier-open list including the parse-error line and its fall back to round 1's rule, the `decisionRuleMarker` variants by round, the judge stencil rewording, and the `PATTERN.md` plus `pattern/PATTERN-review-round.md` amendment. Does it leave the CIRCLING bullet and checkpoint unchanged?"
  - "Fidelity, D4, D5 and D6: do cards 6 to 9 carry the budget of three, the six per-segment keys with `modelspec.Parse` validation at load, `fix_start` with its load refusal and the no-fixer-on-failed-reviewer behavior, and the `loom resume` edge with its refusals and the changed `resume` output and brief wording? Is every Constraint, in particular `PATTERN-refusal-way-forward` spec rows and tests and `PATTERN-shuttle-provider-seam`, carried by some card?"
  - "Attack surface: for the new `loom resume` wake edge, `fix_start: after-review`, and the carry-over write into the decision record, does the card's `**Intent:**` state what it can now skip or let through and what bounds it (D1, D2, D5, D6 each state a Bound)?"
  - "Card order: cards 2 and 3 depend on card 1's `CarryOver` type, and card 4's consumers and card 5's `PATTERN-review-round` amendment depend on the carry-over existing. Does any card use a symbol a later card creates, and does the sequence match the shared decision `pattern-amendment-lands-with-the-rule`?"
  - "Granularity and test economy: cards 1 to 3 and cards 7 to 9 each plan new tests. Do any duplicate an existing test's coverage, or add one test per symbol rather than per public-surface behavior? Name the existing test in any finding."
  - "Verify coverage: does `## verify:` run the tests of every package the cards target, including `internal/discussionparser`, `internal/shedadapters`, `internal/burlerengine`, `internal/loomengine`, `internal/loomcli` and `internal/loomrecipe`, and is it in the template's section order?"
  - "`Custom` and `ImpactSummary`: does any card use `Custom` where a multi-label combination of the other types fits, and does each `Edit` or `Delete` card's `ImpactSummary` carry a blast-radius conclusion rather than a restatement of `Intent`?"
---

Seed focus only; the rubric's `Do not flag` list applies to every entry.
