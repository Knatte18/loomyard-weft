# Card 20 — overview-without-card-index

**Create:**
- `plan:internal/planparser#OverviewWithoutCardIndex` -> `func OverviewWithoutCardIndex(content []byte) ([]byte, error)`

**Edit:**
- `internal/planparser/parse.go`
- `internal/planparser/doc.go`
- `internal/planparser/parse_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Give webster a sole-parser way to compare `00-overview.md` with its Card Index left out (`PATTERN-sole-parsers`), which card 21 hashes to accept an index-only overview change (D7).
`OverviewWithoutCardIndex` returns the overview's bytes with the Card Index section's lines removed: the `## Card Index` heading line through the line before the next `## ` heading, or to the end of the file when none follows, the same bounds `splitFraming` reads the index lines from.
The two share one bounds helper in `parse.go`, so the index the parser builds cards from and the index this function cuts can never differ.
Everything else, frontmatter, title, framing and every later section, is returned byte-for-byte; a missing `## Card Index` heading is the error `splitFraming` already returns.
`parse_test.go` gains `TestOverviewWithoutCardIndex`, one table test for which `test-budget.yaml` raises `internal/planparser` by one: an overview whose index lines change while the rest is fixed yields equal output, a change to the frontmatter, the framing, `## verify:`, `## Shared Decisions` or `## Rename mechanic` yields different output, an index at the end of the file is cut to the end, and a missing heading errors.
The package doc lists the function among the parser's exported reads.

**ImpactSummary:** Adds one exported function and moves splitFraming's index bounds into a shared helper; ParsePlan's results are unchanged.
