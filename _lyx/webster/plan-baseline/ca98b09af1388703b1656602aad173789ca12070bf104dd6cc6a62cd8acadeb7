# Card 12 — answer-symbols-read-the-classifier

**Create:**
- `plan:internal/planglyph#partitionedAnswers` -> `func partitionedAnswers(worktreeRoot string, answers map[string]quarry.ResolveResult) map[string]bool`

**Edit:**
- `internal/planglyph#answerSymbols` -> `func answerSymbols(r quarry.ResolveResult, partitioned bool) (symbols []quarry.Symbol, readable bool)`
- `internal/planglyph#buildEditRegions` -> `func buildEditRegions(edits []editTarget, index map[string]quarry.ResolveResult, partitioned map[string]bool) ([]editRegion, []Finding)`
- `internal/planglyph#LaterDeleteReferences`
- `internal/planglyph#planGatePass`
- `internal/planglyph#coverageSubjects` -> `func coverageSubjects(plan *planparser.Plan, lang glyph.Language, worktreeRoot string, answers map[string]quarry.ResolveResult, partitioned map[string]bool) ([]coverageSubject, error)`
- `internal/planglyph#coveredByTarget` -> `func coveredByTarget(plan *planparser.Plan, lang glyph.Language, subject coverageSubject, file string, line int, answers map[string]quarry.ResolveResult, partitioned map[string]bool) bool`
- `internal/planglyph#callerCoverageFindings` -> `func callerCoverageFindings(plan *planparser.Plan, lang glyph.Language, worktreeRoot string, answers map[string]quarry.ResolveResult, partitioned map[string]bool, loader typesLoader) ([]Finding, error)`
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `internal/planglyph/deleteorder_test.go`
- `internal/planglyph/glyphchain_test.go`

**Uses:**
- `plan:internal/planglyph#classifyAmbiguity`

**Intent:** Let every pass that resolves a member into its declarations see a partitioned member's candidates (decision D6), classifying each ambiguous answer once per pass.
`partitionedAnswers` calls `classifyAmbiguity` once for each `ambiguous` answer in a pass's answer map and returns the targets it reports partitioned, so each candidate file is read once per pass however often the pass consults the answer.
`answerSymbols` takes whether its answer is partitioned: for a partitioned `ambiguous` answer it returns the candidates as the declarations, readable; an unpartitioned ambiguous answer still answers no symbols, readable, and an unreadable status still fails closed.
Bound on the weakened guard: candidates stand as declarations only when they provably never coexist in one build environment, under the classifier's cap and its fallback to not partitioned on any read or parse failure, as card 10 states; this makes the plan-gate passes check more declarations, never fewer.
`LaterDeleteReferences` and `planGatePass` each build the map once from their own `worktreeRoot`, right after `resultByTarget`; `planGatePass` hands it to `callerCoverageFindings`, which hands it to `coverageSubjects`, `buildEditRegions` and `coveredByTarget`, and each `answerSymbols` call reads its target's entry.
So a partitioned Delete target is checked for later-card references, a partitioned member resolves into its files for `redundant-file-target`, `resign-interface-method` and caller coverage, and a partitioned Edit target's spans are searched, with no candidate file re-read per reference line.
Update the `delete-before-reference`, `redundant-file-target` and `caller-uncovered` bullets of `doc.go` to say a partitioned member's candidates are its declarations.
In `loom-plan-spec.md`, row 33 (`redundant-file-target`) says a member that resolves `not_found`, ambiguous without being partitioned by build constraints, or unreadably is skipped.

Tests: add a row to `TestLaterDeleteReferences` (`deleteorder_test.go`): a Delete target declared in a `//go:build linux` file and a `//go:build !linux` file, still referenced by a later card's Edit code, raises `delete-before-reference`.
Add a case to `TestGlyphChain_RedundantFile` (`glyphchain_test.go`), with the partitioned declarations written into the copied fixture through `writeFixtureFiles`: a card listing one of the member's files beside the partitioned member is `redundant-file-target`.

**ImpactSummary:** Five unexported re-signs whose callers all sit in deleteorder.go, plangate.go and callercoverage.go, no test calling any of them directly; a partitioned member now participates in the plan-gate passes instead of being skipped.
