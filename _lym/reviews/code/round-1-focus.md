---
round: 1
focus:
  - "internal/loomshed/citations.go and the discussion callers in gates.go: does the path rule (D3) and symbol rule (D4) match the decision record's token shapes, exclusions, skipped regions and ` (new)`/` (removed)` markers, and is a quarry outage returned as a wrapped `planindex.ErrQuarryUnavailable` rather than a pass or a finding?"
  - "D1 and D2 callers (`ValidateLoomDiscussion`, `ValidateDarnDiscussion`): do both run the id and acceptance checks beside `Validate` without changing `lym loom decision add`, and do they skip the cases the record names (missing record, missing acceptance section, `Added after Discussion` entries)?"
  - "internal/planglyph/index.go, resolve.go and internal/planindex/planindex.go: does the lifted `glyph-not-found` policy leave the per-card resolution unchanged, and does the new method report declaring files as D6 and D8 need?"
  - "internal/loomshed/seamtest.go, testtier.go and docgo.go: do the seam-test (D5, D6), tier-statement (D7) and `doc.go` (D8) checks fire on exactly the card shapes the record states, and do their findings name the item and the way forward?"
  - "Where the plan checks run (D9) and the docs the change owes: spec files, stencil pointers, CLI help, `doc.go` of each changed package and the PATTERN line (D10, D11); do they land in the same diff and match the record's text?"
  - "Test economy in the new test files (`citations_integration_test.go`, `gates_test.go`, `envkit_test.go`): does each new test cover a behavior no existing test covers, and is any behavior the diff adds left without coverage?"
---

Seed focus drawn from the decision record's scope and the diff's file list; no round has been judged.
