---
round: 1
ledger:
  - key: recipe-cell-profile-golden-gap
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: golden-test-doc-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW findings.
`recipe-cell-profile-golden-gap` (F1) asks for a golden row pinning an empty recipe with a pipe-bearing profile.
`golden-test-doc-comment-stale` (F2) asks for the `TestRenderReadmeGolden` doc comment to describe the profile suffix and `tapestry` default.
Neither gates; both fixes are local and already specified by the finding text.
