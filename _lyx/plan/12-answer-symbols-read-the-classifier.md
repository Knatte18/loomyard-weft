# Card 12 — answer-symbols-read-the-classifier

**Edit:**
- `internal/planglyph#answerSymbols` -> `func answerSymbols(worktreeRoot string, r quarry.ResolveResult) (symbols []quarry.Symbol, readable bool)`
- `internal/planglyph#buildEditRegions` -> `func buildEditRegions(worktreeRoot string, edits []editTarget, index map[string]quarry.ResolveResult) ([]editRegion, []Finding)`
- `internal/planglyph#LaterDeleteReferences`
- `internal/planglyph#planGatePass`
- `internal/planglyph#coverageSubjects`
- `internal/planglyph#coveredByTarget` -> `func coveredByTarget(plan *planparser.Plan, lang glyph.Language, worktreeRoot string, subject coverageSubject, file string, line int, answers map[string]quarry.ResolveResult) bool`
- `internal/planglyph#callerCoverageFindings`
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `internal/planglyph/deleteorder_test.go`
- `internal/planglyph/glyphchain_test.go`

**Uses:**
- `plan:internal/planglyph#classifyAmbiguity`

**Intent:** Let every pass that resolves a member into its declarations see a partitioned member's candidates (decision D6).
`answerSymbols` returns, for an `ambiguous` answer `classifyAmbiguity` reports partitioned, the candidates as the declarations, readable; an unpartitioned ambiguous answer still answers no symbols, readable, and an unreadable status still fails closed.
`buildEditRegions`, `LaterDeleteReferences`, `planGatePass`, `coverageSubjects`, `coveredByTarget` and `callerCoverageFindings` thread `worktreeRoot` to it, so a partitioned Delete target is checked for later-card references, a partitioned member resolves into its files for `redundant-file-target`, `resign-interface-method` and caller coverage, and a partitioned Edit target's spans are searched.
Update the `delete-before-reference`, `redundant-file-target` and `caller-uncovered` bullets of `doc.go` to say a partitioned member's candidates are its declarations.
In `loom-plan-spec.md`, row 33 (`redundant-file-target`) says a member that resolves `not_found`, ambiguous without being partitioned by build constraints, or unreadably is skipped.

Tests: add a row to `TestLaterDeleteReferences` (`deleteorder_test.go`): a Delete target declared in a `//go:build linux` file and a `//go:build !linux` file, still referenced by a later card's Edit code, raises `delete-before-reference`.
Add a case to `TestGlyphChain_RedundantFile` (`glyphchain_test.go`), with the partitioned declarations written into the copied fixture through `writeFixtureFiles`: a card listing one of the member's files beside the partitioned member is `redundant-file-target`.

**ImpactSummary:** Three unexported re-signs whose callers all sit in deleteorder.go, plangate.go, callercoverage.go and two test files; a partitioned member now participates in the plan-gate passes instead of being skipped.
