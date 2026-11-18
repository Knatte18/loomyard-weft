No findings.

- F1, glyph literals: `soloSession.table` appears once in `.go` and `.md` files, as the `soloSessionHolder` constant at `seats.go:124`.
- F3, callers of the description reader: `landingSummary` has no reference left; `completeSummary` has two callers, each parsing first and wrapping with its own text.
- F4, row-entry readers: `readRowAttempts` has two callers, both resolving the entry first.
- F2, parent-holds reads: `parentHoldsWork` is called at `finalize.go:331`, `:342` and `:1265`; the second read feeds both `landCheck` and `syncParent`; `doc.go:209` stays true.
- F5, optional seams: the new row loops over `optionalPullRequestSeams`, so it covers all four names.

Calls made:
1. `grep soloSession.table` over `.go` and `.md` files: F1 member search.
2. `grep landingSummary|completeSummary|summaryparser.Parse(` over `internal/`: F3 caller search.
3. `grep readRowAttempts|RowEntry` over `internal/`: F4 caller search.
4. `grep parentHoldsWork|ParentHoldsTaskWork` over non-test files: F2 read sites; the whole-repo list was missing from context.
5. `grep` over the `PullRequestDeps` block and `deps.go` for the optional-seam names: F5 doc and field set; context held only part of `deps.go`.
