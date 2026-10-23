# Card 1 — plandag additive guess

**Edit:**
- `tools/plandag#main`
- `tools/plandag#run` -> `func run(planDir, outDir, dotBinary string, additiveGuess bool, stdout io.Writer) error`
- `tools/plandag#buildCardGraph` -> `func buildCardGraph(plan *planparser.Plan, additiveGuess bool) cardGraph`
- `tools/plandag#dependencyKind` -> `func dependencyKind(earlier, later map[string]mark, plan *planparser.Plan, additiveGuess bool) edgeKind`
- `tools/plandag#renderMarkdown` -> `func renderMarkdown(g cardGraph, guessed *cardGraph, planDir string) string`
- `tools/plandag#TestRun_WritesMatrixLevelsAndReducedGraph`
- `tools/plandag#TestRun_CSharpMemberGlyphsOverlapByOwnerPrefix`
- `tools/plandag#TestRun_UnparseablePlanFailsWithoutOutput`

**Create:**
- `plan:tools/plandag#additiveGuess` -> `func additiveGuess(column string) bool`
- `plan:tools/plandag#TestRun_AdditiveGuessReportsBothCriticalPaths` -> `func TestRun_AdditiveGuessReportsBothCriticalPaths(t *testing.T)`

**Uses:**
- `tools/plandag#writePlan`

**Intent:**
D5: the measurement comes first, as a `-additive-guess` flag on `tools/plandag`.

`main` registers `-additive-guess` (a bool, default false) and passes it to `run`.
`run` always builds the plain graph with `buildCardGraph(plan, false)` and writes the DOT file from it, unchanged.
With the flag set it also builds `buildCardGraph(plan, true)` and passes it to `renderMarkdown` as `guessed`; without it `guessed` is nil and the report is byte for byte today's.

`additiveGuess(column)` reports whether a matrix column is guessed additive: a path or self glyph whose file is a `doc.go`, anything under `docs/`, `PATTERN.md`, anything under `contracts/specs/`, and a member glyph whose last dotted name segment starts with `wire` or is `addWeftVerbs`, so `internal/loomcli#loomCLI.wire` is guessed additive.
It reads the column through `planparser.ParseRef` and `planparser.PathEntryPath`, never a local glyph grammar (PATTERN-glyph-conversion-chokepoint).
`buildCardGraph` with `additiveGuess` true hands the flag to `dependencyKind`, which skips a change-against-change pair of overlapping columns when either column is guessed additive; a change-against-use pair still makes an edge, since a read of added content stays ordered.

`renderMarkdown` with a non-nil `guessed` adds, after the summary line, one line naming the critical path and the best speedup (cards divided by critical path, two decimals) without the guess and with it, such as `Critical path 9 cards, best speedup 2.11x; with the additive guess 5 cards, best speedup 3.80x.`; the rest of the report stays the plain graph's.
The package comment in `main.go` documents the flag, the guessed set, and that the guess never changes the DOT file.

What the flag lets through: it only changes the report; nothing in the run reads plandag.

Tests: the three existing `run` tests pass `false` and keep their exact expectations.
`TestRun_AdditiveGuessReportsBothCriticalPaths` writes a plan whose cards 1 and 2 both `**Edit:**` `docs/x.md` and card 3 edits `internal/a#T` that card 2 also edits, runs with the flag, and asserts the report's exact speedup line (critical path 3 without, 2 with the guess), and that a member column `internal/z#loomCLI.wireThings` shared by two editing cards is dropped as well, while a shared `internal/a#T` keeps its edge.
The same plan's further card pairs each share one more guessed class, `internal/q/doc.go`, `PATTERN.md`, `contracts/specs/x.md` and `internal/f#fabricCLI.addWeftVerbs`, chained so that any one pair kept as an edge would lengthen the guessed critical path; the asserted speedup line's guessed critical path therefore fails if any class is not guessed.

**ImpactSummary:** one tool package; `run`'s only production caller is `main`, and its three tests are re-signed on this card; no lyx code imports `tools/plandag`.
