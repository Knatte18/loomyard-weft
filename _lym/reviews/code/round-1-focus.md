---
round: 1
focus:
  - "internal/lenses/template.yaml `code-scope`: does its wording carry both halves of the board entry (flag a change that traces to no decision; flag a decision no change builds) without dropping or weakening either?"
  - "internal/lenses/template.yaml `code-scope`: does it name the decision-id source correctly (`### D<n>` headings of `## Decisions`, `Added after Discussion` entries) against internal/discussionparser/decisionids.go, and the fallback to the board entry's goal points when the record has no ids?"
  - "internal/lenses/sets/csharp.yaml review-code fan: does the layer's list stay a valid superset of the base fan with `code-scope` added in the base order, and does the csharp layer's other fans and the template's `review-code` fan agree?"
  - "internal/lenses/config_test.go: do the changed counts and the `remove:` list in the empty-fan subtest still match the seeded template, and does the new `code-scope` phrase assertion add coverage beyond the existing lens-phrase table rather than pinning static prose?"
  - "Scope: does the diff add anything beyond the `code-scope` lens and its fan wiring, such as docs or rule documents the repository's constraints name for a new lens that did not land in the same diff?"
---

Seed focus for round 1.
The answer key is the decision record alone: its `## Goal` holds the board entry, and the other sections are empty.
