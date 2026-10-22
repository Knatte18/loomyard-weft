# Card 8 — multi-alphabet-answers

**Create:**
- `plan:internal/planglyph#crossAlphabetDetail` -> `func crossAlphabetDetail(r quarry.ResolveResult) (string, bool)`

**Edit:**
- `internal/planglyph#statusFindings`
- `internal/planglyph#createFindings`
- `internal/planglyph#doneCheckVerdicts`
- `internal/planglyph#classifyAmbiguity`
- `internal/planglyph#TestClassifyAmbiguity`
- `plan:internal/planglyph#TestGlyphChain_CSharp`

**Uses:**
- `plan:internal/planglyph#writeLanguagePlan`
- `internal/planglyph#writeFixtureFiles`
- `internal/planglyph#createCard`
- `internal/planglyph#deleteCard`

**Intent:** Read quarry `v0.4.0`'s multi-alphabet answers (D6).

`crossAlphabetDetail` reports whether an answer is quarry's answer for a self glyph two alphabets both match, `ambiguous` with no candidates and a non-empty `Languages` list, and renders its detail: the languages, and quarry's way out, name the file, such as `Glyphchain.Sharp/Foo.cs#`, or a member.
Every reader of an ambiguous answer that renders candidates uses it in place of the empty candidate list, so no detail lists nothing:
- `statusFindings` reports the blocking `glyph-ambiguous` with that detail;
- `createFindings` reports the blocking `create-already-exists` with that detail;
- `doneCheckVerdicts` puts that detail in place of `lookupDescription`'s for `create-not-done`, `delete-not-done` and `rename-not-done`.
A member glyph's cross-alphabet `ambiguous` answer carries candidates, each with its `Language`, and keeps the ordinary candidate-list detail.

`classifyAmbiguity` partitions only a candidate set whose files all end `.go`, under every plan language; a set holding any other file is described and never partitioned, so a `.cs` name such as `Foo_windows.cs` is never read as a build constraint.

Every switch over quarry's `Kind` in LoomYard keeps working with the values `v0.4.0` adds: the only readers, the kick-start pack's var-and-const first-line rule, `renameDeclSource`'s Go var-and-const synthesis and `isInterfaceMethod`, test for named Go kinds and pass every other kind through; card 10 restricts the last two to Go symbols, and its C# Rename rows over a field, a const, an event and a property reach them with the new kinds.

Tests:
- `TestClassifyAmbiguity` gains a row: two candidates in `Foo_windows.cs` and `Foo_linux.cs` are not partitioned, where the same pair of `.go` files is.
- `TestGlyphChain_CSharp` gains rows that write a directory `Glyphchain.Sharp` holding `readme.txt` into the fixture copy, so the self glyph `Glyphchain.Sharp#` resolves `ambiguous` across both alphabets:
  an Edit of it is `glyph-ambiguous` at both gates; a Create of it is `create-already-exists`; and `DoneChecks` over a Delete card naming it is `delete-not-done`.
  Each row asserts the finding's detail names `csharp`, `go` and "name the file".

**ImpactSummary:** three readers of an ambiguous answer gain the cross-alphabet detail and `classifyAmbiguity` a `.go`-only key; a Go-only candidate set partitions exactly as before, and the empty-candidate answer only `v0.4.0` produces is the only new input.
