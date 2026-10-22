# Fixer report — plan, round 2

## Changes

- **F1** (MEDIUM, card 6), fixed differently.
  The resolve half of `languageMismatchFindings` now never fires when any card holds a Create declaration, since its `plan:` handle names new Go code and `collectGlyphTargets` drops every handle before the batched resolve.
  The proposal's exemption for a Rename pair's new side was not made: the pair's old side is a non-handle ref that `collectGlyphTargets` keeps and resolves, so it decides the language, and exempting the new side would let a `go` plan renaming a C# member (`Glyphchain.Sharp#Renderer.Title` -> `plan:Glyphchain.Sharp#Renderer.Name`) through both halves.
  `TestGlyphChain_CSharp` gains the proposed `go` row (Create `plan:shapes#NewThing` with `func NewThing()` beside the path `csharp/Renderer.cs` reports none) and a Rename row that reports `plan-language-mismatch`.
- **F2** (LOW, cards 9, 11, 15), fixed as proposed and widened.
  Card 9: `CanonicalizeHandles` appends the canonical-spelling way forward to a C# Create handle's `handle-name-failed` detail when quarry's reason is `invalid_container` and the head's explicit-interface qualifier differs from its canonical spelling; the `IPair<string,int>.First` row asserts the clause.
  The condition is narrowed to a differing spelling because `invalid_container` is also raised for other container faults (quarry `internal/engine/name.go`), where the advice would not clear the refusal.
  Card 11: `resignHeadMismatch`'s naming-error way forward gains the same clause, since a re-sign arrow head reaches the same quarry check.
  Card 15: the plan-spec bullet says a C# head, a handle's or a re-sign arrow's, spells an explicit-interface qualifier canonically.
- **F3** (LOW, card 9), fixed as proposed.
  `csharpOwner` takes the declared-name position from `declaredNameIndex`, or, when it reports false, from a depth-zero `this` followed by `[`, so an explicit-interface indexer gets the owner `Renderer`; `declaredNameIndex` still reports false for that head, as card 10 relies on.
  `TestGlyphChain_CSharp` gains the Create row `plan:Glyphchain.Sharp#Renderer.IProvider<int>.this[int]` -> `int IProvider<int>.this[int index] { get; }`.
  The following sentence now reads "the qualifier components" and "no declared-name position", so it agrees with the new definition.

## Self-check

- fix-consistency, LOW: card 9's "or in which no declared name is found" contradicted the new declared-name position; fixed as proposed.
- fix-affected-set, LOW: F2's way forward missed re-sign arrows; fixed as proposed in cards 11 and 15.
- fix-proposal, MEDIUM: the Rename-new exemption in card 6 let a wrong-language Rename plan through; fixed as proposed, with a Rename row added.
- fix-proposal, LOW: the `invalid_container` clause fired for container faults other than a non-canonical qualifier; fixed as proposed, by requiring the written qualifier to differ from its canonical spelling.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
