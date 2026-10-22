---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-plan-language-mismatch.md:31
    summary: "card 6's resolve half refuses a go plan whose only Go work is a Create or Rename-new handle beside a .cs path target, a plan D4's bound says is not refused"
    fix: "In card 6's `languageMismatchFindings` paragraph (the one starting `languageMismatchFindings is the resolve half`), add a firing condition: it does not fire when any card declares a `plan:` handle, a Create declaration or a Rename pair's new side, since under `go` such a handle names Go code and the batched resolve never sees handles (`collectGlyphTargets` drops every `plan:` handle). This mirrors the pure half, which already exempts a plan holding a handle. In the same card's Tests, add a `go` row to the `TestGlyphChain_CSharp` list: a Create card with handle `plan:shapes#NewThing` and head `func NewThing()` beside an Edit of the path `csharp/Renderer.cs` reports no `plan-language-mismatch`. The fix changes no card target or Verify line, so every check in the plan spec admits it."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/09-csharp-create-handles.md:38
    summary: "the Create row that pins invalid_container leaves a planner-reachable dead end: a head spelling an explicit-interface qualifier with a space is refused by quarry's text alone, and neither the finding detail nor the planner contract (card 15) names the canonical spelling"
    fix: "In card 9, state that `CanonicalizeHandles` appends to a C# Create handle's `handle-name-failed` detail, when quarry's reason is `invalid_container` and the head writes an explicit-interface qualifier, the way forward: write the qualifier in the head in canonical spelling, with no space after a comma in its type arguments (`IPair<string,int>.First`). Extend the `IPair<string,int>.First` row to assert the detail holds that clause beside quarry's `invalid_container`. In card 15, add one sentence to the `contracts/specs/loom-plan-spec.md` bullet: a head spells an explicit-interface qualifier in the canonical C# spelling of its type arguments. Text-only changes, no target or Verify change."
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/09-csharp-create-handles.md:27
    summary: "csharpOwner finds the interface qualifier only through declaredNameIndex, which reports false for a head holding `this` at depth zero, so an explicit-interface indexer keeps its qualifier in the owner and quarry refuses the owner for its type arguments"
    fix: "In card 9, say `csharpOwner` takes, as the declared-name position for the qualifier run, a depth-zero `this` followed by `[` when `declaredNameIndex` reports false, so `int IProvider<int>.this[int index] { get; }` for the glyph `Renderer.IProvider<int>.this[int]` gives the owner `Renderer`; `declaredNameIndex` itself keeps reporting false for it, which card 10's Rename relies on. Add a Create row to `TestGlyphChain_CSharp`: `plan:Glyphchain.Sharp#Renderer.IProvider<int>.this[int]` with that head canonicalizes to itself, both gates clean. The row needs no fixture member, since Create handles are named without resolving. Text-only change, no target or Verify change."
---

### [MEDIUM] Card 6's resolve half refuses a Go plan whose Go work is only handles

**Location:** `_lyx/plan/06-plan-language-mismatch.md:31`

**Issue:** D4's bound says a Go plan that edits `.cs` fixtures beside any `.go` file is not refused.
Card 6's pure half honours this for handles: it exempts a plan holding a `plan:` handle, "since such a ref may name `.go` code and is the resolve half's to judge".
The resolve half cannot judge a handle.
`collectGlyphTargets` in `internal/planglyph/planglyph.go` drops every `plan:` handle before the batched resolve, so a Create or Rename-new handle yields no `.go` evidence in the answers.
The resolve half then fires on its own trigger "some path target names a `.cs` file" (or a found self glyph whose `Listing` holds a `.cs` file).
A `go` plan that creates `plan:internal/x#NewFunc` in an existing package and edits a `.cs` fixture is therefore refused as `plan-language-mismatch`, and the detail's way forward (`language: csharp`) is wrong for it.
D4 lists only a `.cs` symbol answer as the resolve half's trigger; the path and `Listing` triggers are the card's additions, and they are what make the handle-only plan reachable.
The repository now holds `.cs` fixtures in a Go hub, so this plan shape is not hypothetical.

### [LOW] Card 9's `invalid_container` row pins a refusal that names no way forward

**Location:** `_lyx/plan/09-csharp-create-handles.md:38`

**Issue:** I checked the row against `checkMintedContainer` in quarry v0.4.0 `internal/engine/name.go`.
It fails as asserted: the minted owner carries the canonical component `IPair<string,int>`, and `strings.Contains(d.Decl, "IPair<string,int>.")` is false for the head `int IPair<string, int>.First { get; }`.
The card correctly notes that quarry matches the qualifier as written, which the decision record's D7 does not know.
The result is a refusal a planner reaches by writing the natural spelling with a space.
Its detail is quarry's "declaration mints ... outside the requested container", which does not say the qualifier must be canonical, and card 15's spec bullet does not teach it.
The row proves the failure but not that the planner can recover from it.

### [LOW] `csharpOwner` leaves an explicit-interface indexer's qualifier in the owner

**Location:** `_lyx/plan/09-csharp-create-handles.md:27`

**Issue:** D7 removes the interface qualifier immediately before "the declared member name" for every C# declaration.
An indexer's glyph name is `this`, and quarry's strategy mints the explicit qualifier into the owner for `indexer_declaration` too (`csharpExplicitOwner` in `internal/engine/csharp.go`).
Card 9 finds the name token with `declaredNameIndex`, which by design reports false for a head holding `this` at depth zero, so `csharpOwner` leaves the chain whole.
Quarry's `nameCSharpContainer` then refuses an owner component with type arguments, and a Create handle or re-sign arrow on `int IList<int>.this[int index]` can never bind.
The Rename path rightly needs the false answer; only the qualifier lookup needs the `this` position.

## Rejected

- Card 4's row pair: the `csharp` row discriminates.
  `Glyphchain#` has no dot, so `hasFileExtension` does not read it as a file under `go`; it is a package directory covering `Glyphchain/notes.txt`.
  Under `csharp` it is a namespace with no `UnitPath`, so the guard reports the file only when it parses through the plan.
  A guard still reading the default Go alphabet covers the file and the `csharp` row fails.
  A dotted unit such as `Glyphchain.Sharp#` would be read as a file under `go` and could not discriminate; the card avoids it.
- `declaredNameIndex` yields the member name for the categories the focus names, by tracing the card's rule against heads shaped as quarry's `SignatureCut` returns them.
  `class Box<T>` gives `Box`, `int IProvider<int>.Count { get; }` gives `Count`, `public void Wrap<U>(T item) where U : class` gives `Wrap`, and a finalizer or static constructor reports false by the `~` and `static` rules.
  The generic-type-of-the-member's-name case is D7's stated bound.
- Card 9's other rows and card 10's rows bind as asserted under `checkMintedContainer`.
  The qualifier rows leave `extra` as the qualifier components, found in the head as written; the nested `Outer.Item` row leaves `extra` empty; the property, field, const and accessor-event renames mint ids equal to the asserted handles from heads cut at the declarator.
- No detail in cards 3, 6, 7, 8 or 11 builds a disk path from a glyph unit.
  Card 7 reads the disk only through `PathEntryPath`, which is `glyph.Parse(glyph.Go, ...)` plus `Glyph.UnitPath`, the single exported reading the declared PATTERN change and D3 allow.
  Card 8's detail is generic text with no constructed path.
- The ways forward in cards 3, 7, 8 and 11 clear the refusal for the inputs that reach them.
  Card 6's way forward clears its own pure half and the handle-free resolve half.
- Card 10's `TestRenameCSharpDecl` is not a per-symbol test: the exact derived head is observable at no public surface, and the decision record's acceptance criteria name those heads.
- Card 6's `Listing` evidence in the resolve half is a conservative extension of D4 for the `.go` side and consistent with D4's "only source targets" bound for the `.cs` side, apart from the handle case in F1.

I did not depart from the focus directive.
