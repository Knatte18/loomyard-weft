---
round: 1
focus:
  - "Cards 2, 3 and 4 each edit `contracts/stencils/claims_test.go` with no `Uses` or ordering between them: settle whether the three edits to one table can land in card order without overlapping rows, or whether the batching is a `design` defect."
  - "Card 3's two rubric edits (Plan-Review granularity and test-economy, Webster-Review test-economy): settle whether each new bullet's `**Intent:**` states what the finding can block and the named-covering-test bound (attack-surface question), and whether the Webster-Review case list matches the decision record's D5 cases one for one."
  - "Fidelity of D8's `refusal-spec.md` half: the plan carries it only as a Shared Decision saying no card is needed. Settle whether that claim holds against the file's actual test wording and whether the decision record's acceptance criterion for `refusal-spec.md` is covered by some card."
  - "Fidelity of the forbidden-phrase lists: for each file, settle whether the phrases a card says must be gone match the decision record's lists, and whether any old phrase named in the decision record's lists has no card editing the passage that carries it."
  - "Verify coverage: settle whether the `## verify:` section runs the tests of `contracts/stencils` (the claims table) and `internal/planparser` (the golden fixture the spec's worked example must stay consistent with)."
  - "Card 1's `**Prosa:**` group targets only a spec file with no coverage edit: settle whether the planparser golden fixture's consistency with the worked example is actually protected by what the card says it leaves untouched."
---

Seed focus drawn from the decision record's scope and constraints and the plan's cross-card structure.
No entry restates a gate check, a dependency-edge gap or an `ImpactSummary` omission on a non-`Edit`/`Delete` card.
