---
round: 2
exclude_lenses:
  - board-fit
focus:
  - "In decision-record.md D4, check that the way citation glyphs are resolved names a method that exists or is added by the Decision: every planindex.Index method in internal/planindex/planindex.go takes a *planparser.Plan, and internal/planglyph/resolve.go builds its glyph-not-found status per card. Settle whether the Decision states the new or changed entry point, its inputs, and how an ambiguous answer and a quarry failure (planindex.ErrQuarryUnavailable) resolve."
  - "In decision-record.md D5, check that the reflect scan trigger matches how today's tests call reflection: internal/websterengine/config_test.go, internal/fabricengine/mergevocab_test.go and internal/loomengine/config_test.go call NumField on a reflect.Type, reflect.VisibleFields is a package function, and internal/testkit/envkit/envkit_test.go calls NilSeams unqualified from inside package envkit. Settle whether each such test is caught by the stated trigger."
  - "In decision-record.md D8, which runs in planContextFindings (internal/loomshed/plancontext.go) with no code index, check how a target is classified as test or non-test code when a glyph like internal/loomshed#TestNewDiscussionGate shares its unit with production members. Settle whether the classification rule is stated and consistent with D8's Bound and its acceptance bullet."
  - "In decision-record.md D6 and D8, for every escape hatch (a seam test or doc.go listed under Uses, a later card's Edit or Extends as cover), check that the Decision's Bound states what it lets through and what catches it."
---

Round 1 left four design findings at MEDIUM: the D4 resolution method, the D5 scan trigger, the D8 test classification, and the Uses-listing bound.
`board-fit` ran in round 1 with no findings and its area is not in the focus, so it is excluded.
