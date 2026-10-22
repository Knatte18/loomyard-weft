# Card 6 — plan-language-mismatch

**Create:**
- `plan:internal/planparser#PathEntryPath` -> `func PathEntryPath(ref string) (string, bool)`
- `plan:internal/planparser#checkLanguageMismatch` -> `func checkLanguageMismatch(plan *Plan) []ValidationError`
- `plan:internal/planglyph#languageMismatchFindings` -> `func languageMismatchFindings(plan *planparser.Plan, results []quarry.ResolveResult) []Finding`
- `plan:internal/planparser_test#TestValidate_LanguageMismatch` -> `func TestValidate_LanguageMismatch(t *testing.T)`

**Edit:**
- `internal/planparser#validate`
- `internal/planparser#checkGlyphMalformed`
- `internal/planglyph#resolvePass`
- `internal/planparser_test#TestValidate_GlyphMalformed`
- `internal/planglyph#TestGlyphChain_CSharp`

**Uses:**
- `internal/planglyph#writeLanguagePlan`

**Intent:** Check the plan's language against its files' extensions (D4), with the new blocking finding `plan-language-mismatch`.

`PathEntryPath` is the Go-alphabet path reading of a canonicalized path entry: it parses the ref with `glyph.Parse(glyph.Go, ...)` and returns `Glyph.UnitPath` when the glyph is a self glyph, else false.
It is the one place a ref is read as a path whatever the plan's language, and this card's check and card 7's namespace-pair check are its only callers.

`checkLanguageMismatch` is the pure half, registered in `validate` right after `checkLanguageRecognized`, so both `ValidateFormat` and `Validate` run it; it reads no disk.
Under `go` it fires when the cards' Targets read through `PathEntryPath` name at least one `.cs` file and no `.go` file, and no Target is a member glyph, a `plan:` handle or a self glyph whose `PathEntryPath` reading has no file extension, since such a ref, a package or directory self glyph included, may name `.go` code and is the resolve half's to judge; under `csharp` when they name at least one `.go` file (a root `tool.go` included, which reads as the path `tool.go`) and no `.cs` file, and no Target is a member glyph `ParseRef` parses as C# or a `plan:` handle, since under `csharp` such a ref names C# code.
It never fires under `none` or an unrecognized language.
One plan-level finding, its detail naming the files' language and the way forward: set `language:` to the language of the files, or `language: none` for a plan that changes both.
The package comment's check list in `validate.go` names it.

`languageMismatchFindings` is the resolve half, called by `resolvePass` over its batched resolve answers, under `go` only, and only when no card is done, so it judges the whole plan: at a mid-run dispatch the pending cards alone may hold only the plan's `.cs` refs, and an unattributed finding there is never dropped.
It fires the same finding, blocking and unattributed to a card, when no path target names a `.go` file, no found or multipart answer has a symbol in a `.go` file, no found self glyph's `Listing` holds a `.go` file, and some answer has a symbol whose `File` ends `.cs`, some path target names a `.cs` file, or some found self glyph's `Listing` holds a `.cs` file.
It never fires when any card holds a Create declaration whose head begins with the Go keyword `func`, `type`, `var` or `const`, since such a handle names new Go code and the batched resolve never sees handles (`collectGlyphTargets` drops every `plan:` handle); a Create whose head is not a Go declaration grants no exemption, so a C# plan missing its `language:` key is still refused when it creates a member, and a Rename pair's new-side handle grants none either, since the pair's old side resolves and decides the language.
So a `go` plan whose member glyphs resolve into `.cs` files and none into a `.go` file is refused, as is one whose package or directory self glyph lists only `.cs` files beside `.cs` paths, or whose only Rename old sides resolve into `.cs` files, and a Go plan with `.go` and `.cs` targets, named as paths, as member glyphs or as a Go Create handle beside `.cs` paths, is not.

`checkGlyphMalformed` under `go`, for a ref `glyph.Parse(glyph.CSharp, ...)` accepts, appends to its detail that the ref is a C# glyph and the way forward, `language: csharp`.

What it does not catch, D4's bound: a `go` plan whose only C# refs are namespace self glyphs such as `Glyphchain.Sharp#`; `path-missing` refuses such a ref when nothing exists at that path and no card creates or renames into it, and neither detail names `language: csharp`.

Tests:
- `TestValidate_LanguageMismatch` is one table over in-memory plans: `go` with only `src/Foo.cs` fires; `csharp` with only `internal/x/x.go` fires; `csharp` with only a root `tool.go` fires; `go` with `a.go` and `src/Foo.cs` does not; `go` with the member glyph `internal/x#Foo` and `src/Foo.cs` does not; `go` with the package self glyph `internal/x#` and `src/Foo.cs` does not; `csharp` with `src/Foo.cs` does not; `csharp` with the member glyph `Glyphchain.Sharp#Renderer.Title` and `tool.go` does not; `none` with only `src/Foo.cs` does not; each firing row asserts the detail names `language:` and `language: none`.
- `TestValidate_GlyphMalformed` gains a `go` row whose target `Glyphchain.Sharp#Renderer.Draw(int)` is one `glyph-malformed` whose detail names `language: csharp`.
- `TestGlyphChain_CSharp` gains `go` rows written with `writeLanguagePlan`: an Edit of `Glyphchain.Sharp#Renderer.Title`, which parses as a Go glyph and resolves into `csharp/Renderer.cs`, reports `plan-language-mismatch`, and the row asserts its detail names `language:`; the same card beside `shapes#Func` reports none; the same card beside the path `shapes/shapes.go` reports none; the same card beside the package self glyph `shapes#`, whose found `Listing` holds `shapes/shapes.go`, reports none; an Edit of the directory self glyph `csharp#` alone, whose found `Listing` holds only `csharp/Renderer.cs`, reports `plan-language-mismatch`; a Create card with the handle `plan:shapes#NewThing` and the head `func NewThing()` beside an Edit of the path `csharp/Renderer.cs` reports none; a plan with no `language:` key, one card editing `Glyphchain.Sharp#Renderer.Title` and one creating `plan:Glyphchain.Sharp#Renderer.Name` with the head `public string Name { get; }`, reports `plan-language-mismatch`; a Rename card with the pair `Glyphchain.Sharp#Renderer.Title` -> `plan:Glyphchain.Sharp#Renderer.Name` reports `plan-language-mismatch`, its old side resolving into `csharp/Renderer.cs`; and a two-card `go` plan, card 1 editing `shapes#Func` and card 2 editing `Glyphchain.Sharp#Renderer.Title`, checked by `ValidateDispatch` with card 1 completed, reports none.

**ImpactSummary:** adds one blocking plan check and one resolve-pass finding; no existing Go plan targets `.cs` files alone, and `glyph-malformed` only gains detail text.
