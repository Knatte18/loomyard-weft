# Fixer report — plan, round 1

## Changes

- F1 (BLOCKING), fixed differently in part.
  Card 9: `declaredNameIndex`'s fallback is now the declarator-position rule: the last identifier at depth zero, outside attribute lists and every `()`, `[]` and `<>` pair, before the first depth-zero `(`, `{`, `=`, `;`, `=>`, `:`, `,`, `where` or the end.
  It reports false for an identifier preceded by `~` or by a bare `static`.
  The examples `class Box<T>` gives `Box` and `public void Wrap<U>(T item) where U : class` gives `Wrap` were added.
  Card 10: `renameCSharpDecl` calls `declaredNameIndex` with the symbol's own name and keeps the index only when the token there is that name, so it never takes the fallback.
  `TestRenameCSharpDecl` gains a `~Renderer()` row asserting false.
  Departures from the proposed stop list, with evidence:
  `:` and `,` were added, since without them `class Renderer : IProvider<int>, IDisposable` falls back to `IDisposable` and `public int a, b;` to `b`.
  `[` was left out (self-check, fix-proposal), since a depth-zero `[` stop gives false for `int[] IProvider<int>.Items { get; }`, whose only token before `[` is the keyword `int`.
  Keywords are stated as never being identifiers, and the rule now reports false when the head holds `operator` or `this` at depth zero, since otherwise `public static Box operator +(Box a, Box b)` gives `Box`.
- F2 (BLOCKING), fixed as proposed.
  Card 9's `IPair<string, int>.First` row is now one `handle-name-failed` carrying quarry's `invalid_container`.
  The `csharpOwner` paragraph gains the sentence on `checkMintedContainer` matching the qualifier as written (quarry `v0.4.0` `internal/engine/name.go:399-415`, checked).
- F3 (BLOCKING), fixed as proposed.
  Card 4's pair now uses `Glyphchain#` and `Glyphchain/notes.txt` (`hasFileExtension`, `internal/planparser/normalize.go:124`, checked).
- F4 (MEDIUM), fixed as proposed, then extended in the self-check.
  Card 6's pure half skips an extensionless self glyph under `go`, and a C# member glyph or `plan:` handle under `csharp`.
  Its resolve half counts a found self glyph's `Listing` holding a `.go` file as Go evidence.
  The self-check found that the skip let a `.cs`-only directory self glyph pass both halves.
  The resolve half now also takes as `.cs` evidence a `.cs` path target or a found self glyph whose `Listing` holds a `.cs` file.
  Rows added: `TestValidate_LanguageMismatch` gains `go` `internal/x#` with `src/Foo.cs` (no finding) and `csharp` `Glyphchain.Sharp#Renderer.Title` with `tool.go` (no finding).
  `TestGlyphChain_CSharp` gains `go` `Glyphchain.Sharp#Renderer.Title` beside `shapes#` (no finding) and `go` `csharp#` alone (one `plan-language-mismatch`).
- F5 (MEDIUM), fixed as proposed.
  In card 3 a rejected Create body branches on the `Lang` field of the `*glyph.ParseError` `ParseRef` returns (quarry `v0.4.0` `glyph/errors.go:130`, checked), so no caller outside planparser maps a language string.
  The self-check (fix-affected-set) asked for the `csharp` branch to key on the same field, and it now does.
- F6 (MEDIUM), fixed as proposed.
  In card 3, under `csharp`, a rejected Create body is `handle-name-failed` with the C# parse error and the C# handle spelling, and makes no quarry call.
  The `Bad(Int32)` row asserts that the detail names the spelling.
  Card 10's twin way forward now also says "with canonical parameter types".
- F7 (MEDIUM), fixed as proposed.
  Card 11's C# `resign-head-mismatch` detail now has one way forward per case.
  The other-member row asserts "Rename pair" and the fails-to-name row asserts "correct the head".
- F8 (MEDIUM), fixed as proposed, then refined in the self-check.
  Card 8's detail builds no path.
  It names the languages from `Languages`, and as the way forward a file (a `.cs` file declaring the namespace, or a file of the directory) as a slashed path self glyph, or a member.
  The "name the file" assertion still matches.
- F9 (LOW), fixed as proposed.
  Card 5 adds the Edit targets `CheckFabricVocabularyName`, `brokenFabricVocabularyRule` (re-sign arrow with a `goToolchain bool` parameter) and `TestCheckFabricVocabularyName`, and extends the gating bullet.
  The test row is a `csharp` copy of the existing row `plan:internal/fabricengine_test#Test<Side>Thing`.
  It draws one finding where `go` draws none, since a C# namespace unit has no slash and the trim is otherwise unobservable.
- F11 (LOW), fixed as proposed.
  Card 12's `resign-not-done` detail names `lyx webster rebaseline --card <card number>`, and the recover-batch step comes from websterengine's not-done way forward (`internal/websterengine/batchfail.go:45`).
- F12 (LOW), fixed as proposed.
  Card 7's detail adds a way forward: move the pair to a card after the one that creates the old side.
- F13 (NIT), fixed as proposed.
  Card 13's Intent gains a sentence saying `boundary.go`'s file header names the re-sign boundary, with no file self glyph listed.

`lyx loom validate-plan` reports `ok` after the changes, with only the existing informational `create-new-unit` finding.

## Self-check

- fix-consistency, MEDIUM: card 6's new self-glyph skip lets a `.cs`-only directory plan through. Merged with fix-proposal's MEDIUM below. Fixed with own change: `.cs` evidence in the resolve half from a `.cs` path target or a found `Listing`, plus a `csharp#` row, rather than only restating the bound.
- fix-consistency, LOW: card 5's ImpactSummary said "six" checks. Fixed as proposed, reworded to name no count.
- fix-consistency, LOW: card 10's way forward lacked "with canonical parameter types". Fixed as proposed.
- fix-claims: no findings.
- fix-proposal, MEDIUM: a depth-zero `[` stop breaks array-typed heads. Fixed as proposed: `[` dropped from the stop list, with the `int[] IProvider<int>.Items` example.
- fix-proposal, MEDIUM: card 6's skip opens a hole (merged with fix-consistency's MEDIUM above). Fixed as proposed.
- fix-proposal, LOW: card 8's way forward was singular and dropped the directory reading. Fixed as proposed.
- fix-affected-set, LOW: card 3's `csharp` branch keyed on the plan language. Fixed as proposed: both branches key on `ParseError.Lang`.

## Disputed

- F10, premise false.
  The finding says card 5's `csharp` row for a Delete of `Glyphchain.Sharp#Renderer.Title` "may pass with the `callerCoverageFindings` gate removed".
  Without the gate, `coverageSubjects` (`internal/planglyph/callercoverage.go:211-276`) takes that subject, since the glyph parses under `csharp` and resolves found.
  It then calls `packageNameOfFile` on the declaring file `csharp/Renderer.cs` (`internal/planglyph/deleteorder.go:468`, `parser.ParseFile(..., parser.PackageClauseOnly)`).
  That call fails on C# source and returns an `ErrQuarryUnavailable`-wrapped error, so the plan gate errors and the row's clean expectation fails.
  The row discriminates the gate as written.
  The proposed replacement would also not discriminate.
  Under `csharp`, card 2's `ParseRef` returns the C# error for a non-self Go member such as `nested/inner#Help` ("a ref only `glyph.Parse(glyph.Go, ref)` accepts is returned only when it is a self glyph"), so `coverageSubjects` would never see that subject.

## Deferred

None.
