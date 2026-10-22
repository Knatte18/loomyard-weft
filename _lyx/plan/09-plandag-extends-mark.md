# Card 9 — plandag Extends mark

**Create:**
- `plan:tools/plandag#markExtends` -> `const markExtends mark = 'E'`
- `plan:tools/plandag#TestRun_ExtendsCellsDropExtendsEdges` -> `func TestRun_ExtendsCellsDropExtendsEdges(t *testing.T)`

**Edit:**
- `tools/plandag#buildCardGraph`
- `tools/plandag#dependencyKind` -> `func dependencyKind(earlier, later planparser.Card, earlierCells, laterCells map[string]mark, labelled map[string]bool, plan *planparser.Plan, additiveGuess bool) edgeKind`
- `tools/plandag#renderMarkdown`
- `tools/plandag#main`

**Uses:**
- `plan:internal/planparser#CardRefKind`
- `plan:internal/planparser#DependencyEdge`
- `tools/plandag#writePlan`
- `plan:tools/plandag#additiveGuess`

**Intent:**
D4 and D5 in the measurement tool: plandag draws Extends cells as `E` and decides each overlapping pair with the shared pair rule, so the tool and Webster cannot drift.

`buildCardGraph` marks each cell from `planparser.CardRefKind` over the card's raw ref: `C` for a change, `E` for an Extends target, `U` for a use, keyed by `columnKey` as today; it collects `labelled`, the columns any card marks `E`.
`dependencyKind` maps each mark of an overlapping column pair (`columnsOverlap`, unchanged) to a `planparser.RefKind` and asks `planparser.DependencyEdge(earlier, …, later, …)`; a pair that makes an edge is `edgeChange` when neither side is a use and `edgeUse` otherwise.
With `additiveGuess`, the guess applies only to a column not in `labelled`, so a labelled ref keeps its label.
Its one caller, `buildCardGraph`, passes the two cards, their cells and `labelled`.

`renderMarkdown`'s matrix and dependency legends name `E` (the card only adds to the ref; two `E` cells on one ref make no edge unless both cards fence a line for one PATTERN entry ID).
The package comment on `main` describes the `E` mark and that the edge rule is `planparser.DependencyEdge`.

`TestRun_ExtendsCellsDropExtendsEdges` writes a plan whose cards 1 and 2 both name `docs/x.md` under `**Extends:**` and card 3 names it under `**Edit:**`, runs without the guess and asserts the matrix shows `E`, `E`, `C`, that cards 1 and 2 have no edge, and that card 3 depends on both; a second run with `-additive-guess` on a plan whose cards 4 and 5 change `docs/y.md` while card 6 labels it `**Extends:**` asserts the labelled column keeps the rule's edges rather than the guess.
The existing `run` tests carry no Extends label and keep their exact expectations.

**ImpactSummary:** confined to `tools/plandag`; `dependencyKind`'s only caller is `buildCardGraph`, edited here; reports of plans without Extends labels are unchanged.
