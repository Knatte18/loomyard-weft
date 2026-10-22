---
round: 1
focus:
  - "internal/lenses/template.yaml: does each merged lens (`coverage`, `row-validity`, `forced-edits`) keep every check of the lenses it replaces, as the board entry asks, or does a procedure step from `criteria-coverage`, `branch-coverage`, `row-reachability`, `test-economy`, `affected-set` or the nil-seam check get dropped?"
  - "internal/lenses/template.yaml: does any remaining fan, stencil, doc or lens list outside the three changed files still name a deleted lens (`criteria-coverage`, `branch-coverage`, `row-reachability`, `test-economy`)? Settle with grep across the repo, excluding `docs/research/`."
  - "internal/lenses/config_test.go: is the `affected-set` to `forced-edits` perch-parity mapping limited to that one pair, and do the pinned-sentence checks still pin the sentences the new lens texts contain?"
  - "internal/loomengine/plan_test.go: does the `generic` lens in the redefining row still exercise the same behavior the `test-economy` row did?"
  - "_lyx/landing/summary.md: check its claims against the diff, notably the lens order, the emphasis line on each lens, and the `forced-edits` `_test` package detail."
---

Seed focus from the summary and commit stat; the diff itself is the subject.
