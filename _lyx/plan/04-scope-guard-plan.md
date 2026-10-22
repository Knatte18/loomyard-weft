# Card 4 — scope-guard-plan

**Edit:**
- `internal/planglyph#ScopeGuard` -> `func ScopeGuard(plan *planparser.Plan, cards []planparser.Card, delta quarry.GitDeltaAnswer) []Finding`
- `internal/planglyph#symbolFilesCovered` -> `func symbolFilesCovered(plan *planparser.Plan, cards []planparser.Card, files []string) bool`
- `internal/planglyph#targetsCover` -> `func targetsCover(plan *planparser.Plan, cards []planparser.Card, path string) bool`
- `internal/planglyph#batchDelta.ScopeGuard` -> `func (d batchDelta) ScopeGuard(plan *planparser.Plan, cards []planparser.Card) []Finding`
- `internal/planindex#Delta.ScopeGuard`
- `internal/websterengine#postBatchChecks`
- `internal/websterengine_test#fakeDelta.ScopeGuard` -> `func (d fakeDelta) ScopeGuard(plan *planparser.Plan, cards []planparser.Card) []planindex.Finding`
- `internal/planglyph#TestScopeGuard_SymbolCoverage`
- `internal/planglyph#TestScopeGuard_FileLevelCoverage`
- `internal/planglyph#TestScopeGuard_DeletedAndModifiedAlsoChecked`
- `internal/planglyph#TestScopeGuard_EmptyDeltaNoFindingsNoPanic`
- `internal/planglyph#TestScopeGuard_RenamedPairOutsideTargetsIsReported`
- `internal/planglyph#TestScopeGuard_RenamedPairInsideTargetsIsNotReported`

**Intent:** `ScopeGuard` receives the plan (D3), so its `planparser.RefMember` and `planparser.RefScope` calls read the plan's alphabet through `ParseRef` rather than the default Go alphabet a nil plan gives.
`symbolFilesCovered` and `targetsCover` take the plan and pass it to `RefMember` and `RefScope` in place of nil.

The interface method `planindex.Delta.ScopeGuard` gains the leading `plan *planparser.Plan` parameter, so its implementations `batchDelta.ScopeGuard` and the test fake `fakeDelta.ScopeGuard` take it and forward it, and `postBatchChecks` passes `in.Plan`.

Under `go` every verdict is today's.
Under `csharp` a C# member glyph target is a member, never a file or package scope, so it covers only its own symbol, and a C# namespace self glyph covers no file, per D3's bound.

Tests: every existing `ScopeGuard` test passes a `go` plan, and `TestScopeGuard_FileLevelCoverage` gains a pair of rows whose only target is `Glyphchain.Sharp#` while the delta changes the file `Glyphchain.Sharp/notes.txt` with no symbol in it.
Under `go` the target is the package directory `Glyphchain.Sharp`, which covers the file, so there is no finding; under `csharp` it is a namespace with no scope, so the file is one `scope-outside-plan`.
The `csharp` row fails if the guard still reads its targets with the default Go alphabet.

**ImpactSummary:** the scope guard's exported signature and its one interface method change, with both implementations and the one production caller in `postBatchChecks` on this card; the guard stays informational.
