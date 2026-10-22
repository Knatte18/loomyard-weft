# Fixer report — plan, round 2

An earlier fixer session of this round had already applied the three findings' changes to the cards (its report is `round-2-fixer-report-20261022T073815Z.md`).
This session checked those changes against the review and the code, kept them, ran the self-check once and fixed what it found.

## Changes

- **F1** (MEDIUM, card 6), fixed differently.
  The resolve half of `languageMismatchFindings` never fires when any card holds a Create declaration whose head begins with the Go keyword `func`, `type`, `var` or `const`, since such a handle names new Go code and `collectGlyphTargets` drops every `plan:` handle before the batched resolve.
  Two parts of the proposal were not made.
  The exemption for a Rename pair's new side was left out: `collectGlyphTargets` keeps the pair's old side (`add(p.Old)`, `internal/planglyph/planglyph.go:451`) and resolves it, so the old side decides the language, and exempting the new side would let through a `go` plan that renames a C# member (`Glyphchain.Sharp#Renderer.Title` -> `plan:Glyphchain.Sharp#Renderer.Name`).
  The exemption is narrowed to Create handles with a Go head; the proposed blanket exemption for every Create declaration was left out, because it would let through a C# plan missing its `language:` key, which defaults to `go` per cards 2 and 5, whenever that plan creates a C# member.
  `TestGlyphChain_CSharp` gains the proposed `go` row (Create `plan:shapes#NewThing` with `func NewThing()` beside the path `csharp/Renderer.cs` reports none), a Rename row that reports `plan-language-mismatch`, and a row with no `language:` key whose C# Create beside an Edit of `Glyphchain.Sharp#Renderer.Title` reports `plan-language-mismatch`.
  The card's summary sentence now names the refused Rename case and says "Go Create handle".
- **F2** (LOW, cards 9, 11, 15), fixed as proposed and widened.
  Card 9: `CanonicalizeHandles` appends the canonical-spelling way forward to a C# Create handle's `handle-name-failed` detail when quarry's reason is `invalid_container` and the head's explicit-interface qualifier differs from its canonical spelling.
  The way forward quotes the canonical qualifier `glyph.CanonicalCSharpType` gives, not a fixed "no space after a comma" rule.
  The `IPair<string,int>.First` row asserts that the detail quotes `IPair<string,int>.`.
  The condition is narrowed to a differing spelling because quarry also raises `invalid_container` for other container faults (`internal/engine/name.go`), where the advice would not clear the refusal.
  Card 11: `resignHeadMismatch`'s naming-error way forward gains the same clause, since a re-sign arrow's head reaches the same quarry check, and `TestGlyphChain_ResignScenarios` gains a row that asserts it (`IPair<string,int>.First` -> `int IPair<string, int>.First { get; }`).
  Card 15: the plan-spec bullet says a C# head, whether a handle's or a re-sign arrow's, spells an explicit-interface qualifier canonically.
- **F3** (LOW, card 9), fixed as proposed.
  `csharpOwner` takes the declared-name position from `declaredNameIndex`, or, when that reports false, from a depth-zero `this` followed by `[`, so an explicit-interface indexer gets the owner `Renderer`.
  `declaredNameIndex` still reports false for that head, which card 10 relies on.
  `TestGlyphChain_CSharp` gains the Create row `plan:Glyphchain.Sharp#Renderer.IProvider<int>.this[int]` -> `int IProvider<int>.this[int index] { get; }`.

## Self-check

- fix-consistency, LOW: card 11's widened naming-error clause had no test row (also raised by fix-affected-set); fixed as proposed, adding the `IPair<string, int>.First` re-sign row, which asserts the canonical qualifier the new clause quotes.
- fix-consistency, LOW: card 6's summary sentence omitted the refused Rename case; fixed as proposed.
- fix-claims: no findings.
- fix-proposal, MEDIUM: the blanket Create exemption hid the mismatch for a C# plan with no `language:` key that creates a member; fixed as proposed, narrowing the exemption to a Create with a Go head and adding the row.
- fix-proposal, LOW: the `invalid_container` advice covered only a space after a comma; fixed as proposed, with cards 9 and 11 quoting the canonical qualifier from `glyph.CanonicalCSharpType`.
- fix-affected-set, LOW: duplicate of the card 11 missing-row finding above.
  Its out-of-scope note was not acted on: a card 10 Rename of a source member whose qualifier is written with a space is neither a finding of this review nor reached by this fix.

## Disputed

None.

## Deferred

None.
