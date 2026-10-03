---
round: 2
focus:
  - "D2 coverage beyond the usual file names: `internal/gitrepo/noforceadd_test.go` enforces CONSTRAINTS.md's Never Force-Add Invariant by reading its own package directory (`os.Getwd` plus `os.ReadDir`) with a token set and a floor. Settle whether some conversion card now targets it and converts it to a shallow scankit walk with its tokens and floor kept verbatim. Then grep `_test.go` files for other source reads (`os.Getwd`, `os.ReadDir`, `os.ReadFile` of a `.go` file, `parser.ParseFile`) in tests that cite a CONSTRAINTS.md invariant, and settle whether each is a target of cards 2–10 or does not enforce an invariant by walking source."
  - "References to names a card removes: card 15 folds `loomRowEngines` into one table, and production `internal/loomrecipe/names.go:16-18` names that variable in a doc comment. Card 11 deletes `cmd/lyx/drift_test.go` and `cmd/lyx/longlist_test.go`, and `cmd/lyx/seamsignature_test.go:7` and `cmd/lyx/sandbox_coverage_test.go:39` name them. Settle whether each referencing file is now a target of the removing card, with an ImpactSummary that matches. Grep for any other symbol or file a card deletes or renames that is still named outside that card's targets."
---

All thirteen round-1 findings are resolved, and both focus entries cover findings that are still open.
