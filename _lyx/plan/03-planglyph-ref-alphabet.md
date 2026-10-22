# Card 3 — planglyph-ref-alphabet

**Create:**
- `internal/planglyph/testdata/glyphchain/csharp/Renderer.cs`
- `plan:internal/planglyph#writeLanguagePlan` -> `func writeLanguagePlan(t *testing.T, language string, cards []string, sections ...plankit.Section) (string, *planparser.Plan)`
- `plan:internal/planglyph#TestGlyphChain_CSharp` -> `func TestGlyphChain_CSharp(t *testing.T)`

**Edit:**
- `internal/planparser#Plan.GlyphLanguage` -> `func (p *Plan) GlyphLanguage() bool`
- `internal/planparser#TestPlanGlyphLanguage`
- `internal/planglyph#CanonicalizeHandles`
- `internal/planglyph#BindHandles`
- `internal/planglyph#DoneChecks`
- `internal/planglyph#resolvePass`
- `internal/planglyph#collectGlyphTargets` -> `func collectGlyphTargets(plan *planparser.Plan) []string`
- `internal/planglyph#DetectDrift`
- `internal/planglyph#planGatePass`
- `internal/planglyph#collectMemberTargets` -> `func collectMemberTargets(plan *planparser.Plan) []string`
- `internal/planglyph#callerCoverageFindings` -> `func callerCoverageFindings(plan *planparser.Plan, worktreeRoot string, answers map[string]quarry.ResolveResult, partitioned map[string]bool, loader typesLoader) ([]Finding, error)`
- `internal/planglyph#coverageSubjects` -> `func coverageSubjects(plan *planparser.Plan, worktreeRoot string, answers map[string]quarry.ResolveResult, partitioned map[string]bool) ([]coverageSubject, error)`
- `internal/planglyph#coveredByTarget` -> `func coveredByTarget(plan *planparser.Plan, subject coverageSubject, file string, line int, answers map[string]quarry.ResolveResult, partitioned map[string]bool) bool`
- `internal/planglyph#LaterDeleteReferences`
- `internal/planglyph#classifyDeleteOrderRef` -> `func classifyDeleteOrderRef(plan *planparser.Plan, ref string) (refShape, glyph.Glyph, string)`
- `internal/planglyph#collectEditTargets` -> `func collectEditTargets(plan *planparser.Plan, cards []planparser.Card) []editTarget`
- `internal/planglyph#laterImportFindings` -> `func laterImportFindings(plan *planparser.Plan, deleting []planparser.Card, regions []editRegion, worktreeRoot string) ([]Finding, error)`
- `internal/planglyph#kickstartPack`
- `internal/planglyph#writeGlyphPlan`
- `internal/planglyph#TestCollectGlyphTargets_DeduplicatesAcrossCards`
- `internal/planglyph#TestKickstartPack`
- `internal/websterengine#postBatchChecks`
- `internal/websterengine#buildKickstartPack`
- `tools/plandag#buildCardGraph`
- `tools/plandag#dependencyKind` -> `func dependencyKind(earlier, later map[string]mark, plan *planparser.Plan) edgeKind`
- `tools/plandag#columnsOverlap` -> `func columnsOverlap(a, b string, plan *planparser.Plan) bool`

**Uses:**
- `internal/planglyph#copyGlyphChainFixture`
- `internal/planglyph#writeFixtureFiles`
- `internal/planglyph#createCard`
- `internal/planglyph#assertPlanGate`
- `internal/planglyph#assertDispatch`
- `internal/planglyph#editCard`

**Intent:** Route every plan-ref parse outside `internal/planparser` through `planparser.ParseRef` (D3, D8), so `internal/planglyph`, `tools/plandag` and the kick-start pack get C# glyphs once a plan says `csharp`.

`Plan.GlyphLanguage` keeps only its glyph-enabled answer: it returns true exactly when `planLanguage` answers ok, and no caller receives a language value to parse with.
Its callers read only the bool: `CanonicalizeHandles`, `BindHandles`, `DoneChecks`, `resolvePass`, `DetectDrift`, `planGatePass`, `LaterDeleteReferences`, `kickstartPack`, `postBatchChecks`, `buildKickstartPack` and `buildCardGraph`.
Every `glyph.Parse(lang, ...)` in those packages becomes `planparser.ParseRef(plan, ...)`, so the unexported helpers that took a `glyph.Language` take the plan instead: `collectGlyphTargets`, `collectMemberTargets`, `callerCoverageFindings`, `coverageSubjects`, `coveredByTarget`, `classifyDeleteOrderRef`, `collectEditTargets`, `laterImportFindings`, `dependencyKind` and `columnsOverlap`, with every caller updated on this card.
`DetectDrift` parses its reloaded plan's targets through the reloaded plan.

In `CanonicalizeHandles` a Create declaration's `Lang` is the language of `ParseRef` over the handle's body (the text after `plan:`); a body `ParseRef` rejects keeps today's path, its unit from `planparser.HandleUnit`, with `Lang` `glyph.Go` under `go` and `glyph.CSharp` under `csharp`, so quarry answers it `handle-name-failed` as today.
A re-sign arrow's declaration takes `Unit` and `Lang` from `ParseRef` over the arrow's target.
Under `go` every result is what card 1 left.

The C# fixture `internal/planglyph/testdata/glyphchain/csharp/Renderer.cs` declares namespace `Glyphchain.Sharp` with every shape the later cards name: an interface `IProvider<T>` with property `int Value { get; }`; an enum `Color`; a generic class `Box<T>`; a class `Outer` with a nested class `Item`; and a class `Renderer : IProvider<int>, IDisposable` holding `public const int A = 1, B = 2;`, `private Dictionary<string, int> map;`, `private (int, int) pair;`, `public event EventHandler Changed;`, the accessor event `public event EventHandler Moved { add { } remove { } }`, `public Color Color { get; }`, `public string Title { get; set; }`, `public void Draw(int count)`, `public void Draw(string text)`, `public void Fill(int count)`, the explicit `int IProvider<int>.Value` and the explicit `void IDisposable.Dispose()`.
Every glyph a later card's test names is copied from `lyx quarry glyphs --text internal/planglyph/testdata/glyphchain/csharp` once the file exists.
The Go rows of the glyph-chain tests keep their findings with the file present, since no Go unit in the fixture spells `Glyphchain.Sharp`.

`writeGlyphPlan` becomes `writeLanguagePlan` with `language` set to `go`, and `writeLanguagePlan` writes the plan with the `language:` it is given.

Tests:
- `TestGlyphChain_CSharp` is the C# plan table later cards extend: each row writes a `csharp` plan over a fixture copy and asserts `assertPlanGate` and `assertDispatch`.
  Each row carries the files it writes into the fixture copy (through `writeFixtureFiles`) and the exact finding set of each gate.
  This card's rows, each one Edit card validating clean at both gates: one naming `Glyphchain.Sharp#Renderer.Draw(int)` and `Glyphchain.Sharp#`, which parse as C# and resolve found; one naming the path `csharp/Renderer.cs`, kept off the member's card since `redundant-file-target` refuses a file beside a member declared in it; and one naming `README.md`, which the row writes into the fixture copy, so the path canonicalizes to `README.md#`, skips the path checks and resolves found through quarry.
  A further row is a Create card whose draft handle body `Glyphchain.Sharp#Renderer.Bad(Int32)` `ParseRef` rejects, with head `public void Bad(int count)`: it takes the fallback declaration, unit `Glyphchain.Sharp` from `planparser.HandleUnit` and `Lang` `glyph.CSharp`, which quarry cannot name as a member outside a type, so it is one `handle-name-failed`.
- `TestKickstartPack` gains a `csharp` card whose Edit target `Glyphchain.Sharp#Renderer.Draw(int)` is selected and listed resolved with the fake resolve's declaration span.
- `TestPlanGlyphLanguage` asserts the bool alone, true for `go`, `csharp` and the absent value and false for `none`.
- `TestCollectGlyphTargets_DeduplicatesAcrossCards` calls `collectGlyphTargets(plan)`.

**ImpactSummary:** one exported re-sign, `Plan.GlyphLanguage`, with every production caller across planglyph, websterengine and plandag all on this card; the other re-signs are unexported; Go plans parse as before.
