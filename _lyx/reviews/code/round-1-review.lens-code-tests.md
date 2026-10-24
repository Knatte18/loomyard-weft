```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, per behaviour the test that pins it, then Test-economy findings.

**Per behaviour**

- **Earlier card grows past the line, later card shrinks it back, so one `pattern-card-headroom` finding on the earlier card and no `pattern-headroom` finding:** pinned by `TwoCardsFencingOneIDWhereTheLaterFitsFlagsOnlyTheEarlierCard` (`internal/loomshed/gates_test.go`). It asserts `wantHeadroom` 0, `wantCardHeadroom` 1, and the card `1-first-card`.
- **Growth in a later card is attributed to that card:** pinned by `TwoCardsFencingOneIDWhereTheLaterOverflowsIsFlagged`, which expects the card `2-first-card`.
- **Single card growing past the line:** pinned by `PatternLinesGrowingPastTheLineFromABaseUnderIt` and `TwoLinesOfOneCardWhereTheLaterOverflowsIsFlagged`.
- **A card whose lines net to no growth passes, even with an intermediate line that would grow:** pinned by the existing `TwoLinesOfOneCardWhereTheLaterFitsPass` (`longEntryA` then `sameSizeA`). It now covers the per-card check too, because the default `wantCardHeadroom` is 0. No new row was needed.
- **A removal named in prose by the growing card offsets its growth:** pinned by `RemovalNamedByTheGrowingCardOffsetsItsGrowth`. By my arithmetic it fails if crediting is broken, since the growth alone passes the line and only the named removal brings it back under.
- **A removal named only by a later card flags the earlier growing card:** pinned by `RemovalNamedByALaterCardFlagsTheEarlierGrowingCard`.
- **A removal no card names is dropped before the first card:** pinned by `RemovalNoCardNamesShrinksTheProjectionBeforeTheFirstCard`. It also fails if the unnamed removal were never applied. See finding 2 for what it cannot tell apart.
- **A done card is never projected or flagged:** pinned by `DoneCardIsNeverProjected`. The base is already past the line, so the whole-plan check stays silent. A done card that were projected would still be flagged per card, so the row discriminates.
- **Each finding names its card and the way forward:** pinned by the `f.Card != tt.wantCardHeadroomOn` assertion and the shared `wantDetail` check, which now includes `patternCardHeadroomCheck`.
- **The whole-plan `pattern-headroom` check is unchanged:** pinned by the existing rows, which still assert `wantHeadroom` (`PatternOverviewPastTheLineWithNoRemovesIsFlagged` and the others).
- **`CardNamesPatternEntry` (`internal/planparser/patternentry_test.go`):** pinned by `TestCardNamesPatternEntry`. Its four rows cover prose, a fenced line, a bare ID without backticks, and a longer ID sharing the prefix. It tests the exported function of the package.
- **First-naming-card selection (`slices.IndexFunc` picks the first card that names the ID):** no test pins it. See finding 1.

**Findings**

1. LOW, scope, `internal/loomshed/gates_test.go` (new removal rows): no row has two pending cards naming the same removed ID. The decision record says a removal is applied in the first pending card that names it. A mutation crediting the last naming card, or every naming card, passes every row. `RemovalNamedByTheGrowingCardOffsetsItsGrowth` and `RemovalNamedByALaterCardFlagsTheEarlierGrowingCard` each have only one naming card. A row is needed where card 1 grows and names the ID, and card 2 also names it, and the plan expects no finding.
2. NIT, scope, `internal/loomshed/gates_test.go`, row `RemovalNoCardNamesShrinksTheProjectionBeforeTheFirstCard`: it has one card, so "applied before the first card" and "applied in the first card" give the same result, because `ProjectOverview` applies fenced lines and then removals. The row pins "an unnamed removal is applied somewhere early", not the before-first-card placement. This is acceptable given that equivalence, but the row name overstates what it pins.

**Test economy:** no duplicated coverage, no per-helper test that a public-surface test already covers, and no assertion on stencil prose. The changed doc and stencil text have no assertions.

**Tool calls:** none. The diff and fixtures already in context were enough.
