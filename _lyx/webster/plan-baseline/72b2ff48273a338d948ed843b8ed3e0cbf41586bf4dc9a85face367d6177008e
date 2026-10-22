# Card 2 — planparser-ref-alphabet

**Create:**
- `plan:internal/planparser#ParseRef` -> `func ParseRef(plan *Plan, ref string) (glyph.Glyph, error)`
- `plan:internal/planparser#TestParseRef` -> `func TestParseRef(t *testing.T)`

**Delete:**
- `internal/planparser#parseGlyph`
- `internal/planparser#TestParseGlyph`

**Edit:**
- `internal/planparser#planLanguage`
- `internal/planparser#ParsePlanFrom`
- `internal/planparser#checkLanguageRecognized`
- `internal/planparser#sourceExtensions`
- `internal/planparser#isSourceTarget`
- `internal/planparser#diskPathForRef`
- `internal/planparser#checkGlyphMalformed`
- `internal/planparser#checkResignNotMember`
- `internal/planparser#resignTargetShape` -> `func resignTargetShape(plan *Plan, target string) (shape string, bad bool)`
- `internal/planparser#isFileRenamePair` -> `func isFileRenamePair(plan *Plan, p MovePair) bool`
- `internal/planparser#SymbolRenamePairs`
- `internal/planparser#checkRenamePairShape`
- `internal/planparser#checkProsaSymbolTarget`
- `internal/planparser#checkRedundantPackageTarget`
- `internal/planparser#renameToSideUnits` -> `func renameToSideUnits(plan *Plan) map[string]string`
- `internal/planparser#redundancyOwnerOf` -> `func redundancyOwnerOf(plan *Plan, renameUnits map[string]string, ref string) (redundancyOwner, bool)`
- `internal/planparser#CheckEditTestFile`
- `internal/planparser#RefMember`
- `internal/planparser#refUnit`
- `internal/planparser#CheckFabricVocabularyName`
- `internal/planparser#TestPlanGlyphLanguage`
- `internal/planparser_test#TestCheckFabricVocabularyName`
- `internal/planparser_test#TestValidate_ResignNotMember`
- `internal/planparser_test#TestValidate_RedundantPackageTarget`
- `internal/planparser_test#TestValidate_ProsaSymbolTarget`
- `internal/planparser_test#TestParsePlan_Language`
- `internal/planparser_test#TestValidate_LanguageRecognized`
- `internal/planparser_test#TestReviewExempt`
- `internal/planparser_test#TestSymbolRenamePairs`
- `internal/planparser_test#TestValidate_PathMissing`
- `internal/planparser_test#TestValidate_GlyphMalformed`

**Intent:** Put the alphabet choice for every plan ref in one exported `internal/planparser` helper (D3), and admit `language: csharp` (D2).

`ParseRef` takes the plan and a `#`-shaped ref, or a `plan:` handle's body the caller already stripped, and returns its parsed glyph:
- under a language `planLanguage` rejects (`none`, an unrecognized value) it returns an error and parses nothing;
- under `go` (and the absent value) it is `glyph.Parse(glyph.Go, ref)`, exactly as today;
- under `csharp` a ref `glyph.Parse(glyph.CSharp, ref)` accepts is a C# glyph, a ref both alphabets parse included;
  a ref only `glyph.Parse(glyph.Go, ref)` accepts is returned only when it is a self glyph (a slashed path self glyph such as `src/Foo.cs#`);
  any other ref returns the C# alphabet's parse error.
It adds no grammar of its own: the two `glyph.Parse` calls are the whole of it.
`ParseRef` lives in `glyphref.go`, whose file header is reworded from `parseGlyph`'s delegation to `ParseRef`'s alphabet rule.

`parseGlyph` is deleted, and every caller of it goes through `ParseRef` with the plan instead of a language value: `isSourceTarget`, `diskPathForRef`, `checkGlyphMalformed`, `resignTargetShape`, `isFileRenamePair`, `checkRenamePairShape`, `checkProsaSymbolTarget`, `renameToSideUnits`, `redundancyOwnerOf`, `CheckEditTestFile`, `RefMember` and `refUnit`.
`resignTargetShape` names a self glyph with no `UnitPath`, a C# namespace under `csharp`, "a namespace self glyph" rather than falling through to "a file self glyph".
`CheckFabricVocabularyName` takes a handle's declared name from the `Name` of `ParseRef` over the handle's body, with a C# generic arity suffix stripped, whenever `ParseRef` parses it, and keeps `HandleIdentifier`'s last-dot split only for a body `ParseRef` rejects, so `plan:Ns#Renderer.Paint(System.Text.Rune)` names `Paint`, not `Rune)`.
The helpers that took a `glyph.Language` take the plan instead, so their callers `checkResignNotMember`, `SymbolRenamePairs`, `checkRenamePairShape` and `checkRedundantPackageTarget` pass the plan and keep `planLanguage` only as the glyph-enabled test.

`planLanguage` maps `csharp` to `glyph.CSharp` with ok true, and `Plan.GlyphLanguage` keeps returning what it returns; `internal/planglyph` re-signs it on the next card.
`checkLanguageRecognized` accepts `csharp` beside `go`, `none` and the absent value, and its detail names the three legal values.
`ParsePlanFrom` canonicalizes path-shaped entries through `glyph.Self(glyph.Go, path)` under both `go` and `csharp`, since quarry spells a path only in the Go unit alphabet, and never under `none` or an unrecognized value.
`sourceExtensions` maps `csharp` to `.cs`, so `ReviewExempt` counts a `.cs` Prosa target as source under `csharp`.

Under `csharp`, D3's bound follows with no further code: a slash-free self glyph such as `README.md#` or `Program.cs#` parses as a C# namespace, has no `UnitPath`, and so `card-path-malformed`, `path-missing`, `redundant-package-target` and `RefMember`/`RefScope` skip it, and `ReviewExempt` counts it as a source target.
A pair whose two sides are both self glyphs stays a file pair under `csharp`, so `SymbolRenamePairs` leaves `Program.cs -> App.cs` out; the classification reads no disk.

Tests:
- `TestParseRef` replaces `TestParseGlyph` with one table: under `go` a Go member glyph parses Go and a C# member glyph with a parameter list (`Glyphchain.Sharp#Renderer.Draw(int)`) errors; under `csharp` that member parses as C#, a namespace self glyph `Glyphchain.Sharp#` parses as C#, `README.md#` parses as C# (both alphabets take it), `src/Foo.cs#` parses as Go, a slashed member `a/b#Draw(int)` that neither alphabet takes errors, and a Go-only member `a/b#Func` errors with the C# error; under `none` every ref errors.
- `TestPlanGlyphLanguage` gains the `csharp` row answering `glyph.CSharp`, true.
- `TestParsePlan_Language` gains a `csharp` row whose `Program.cs` target canonicalizes to `Program.cs#`.
- `TestValidate_LanguageRecognized` gains the `csharp` row with no finding.
- `TestReviewExempt` gains a `csharp` plan whose only Prosa target is `src/Foo.cs`, which is not exempt, beside the same plan with `docs/a.md`, which is.
- `TestSymbolRenamePairs` gains a `csharp` plan holding `Program.cs -> App.cs`, which answers nil.
- `TestValidate_PathMissing` gains a `csharp` plan editing a missing `README.md`, with no `path-missing`.
- `TestValidate_GlyphMalformed` gains a `csharp` plan whose `a/b#Draw(int)` target is one `glyph-malformed`.
- `TestValidate_ResignNotMember` gains a `csharp` row whose arrow target `Glyphchain.Sharp#` is one `resign-not-member` whose detail says "a namespace self glyph".
- `TestValidate_RedundantPackageTarget` gains a `csharp` row listing `Glyphchain.Sharp#` beside `Glyphchain.Sharp#Renderer.Title`, which draws no `redundant-package-target`.
- `TestValidate_ProsaSymbolTarget` gains a `csharp` row whose Prosa targets `src/Foo.cs` and `Glyphchain.Sharp#` pass and whose `Glyphchain.Sharp#Renderer.Title` is one `prosa-symbol-target`.
- `TestCheckFabricVocabularyName` gains a `csharp` row whose Create handle `plan:Ns#Renderer.Paint(System.Text.Rune)` is checked under the name `Paint`, so no finding names `Rune)`.

**ImpactSummary:** confined to `internal/planparser`; every re-signed helper is unexported with all callers on this card, and under `go` every check parses exactly as before.
