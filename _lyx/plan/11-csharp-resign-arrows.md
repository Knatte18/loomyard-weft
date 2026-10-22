# Card 11 — csharp-resign-arrows

**Create:**
- `plan:internal/planglyph#nameDeclarations` -> `func nameDeclarations(decls []quarry.Declaration) ([]quarry.NameResult, error)`
- `plan:internal/planglyph#resignName` -> `type resignName struct`
- `plan:internal/planglyph#nameResigns` -> `func nameResigns(plan *planparser.Plan) ([]resignName, []Finding, error)`
- `plan:internal/planglyph#resignCollisionFindings` -> `func resignCollisionFindings(plan *planparser.Plan, names []resignName, answers map[string]quarry.ResolveResult) []Finding`

**Edit:**
- `internal/planglyph#CanonicalizeHandles`
- `internal/planglyph#resignHeadMismatch`
- `internal/planglyph#resolvePass`
- `internal/planglyph#chokepointGuardedFuncs`
- `internal/planglyph#TestChokepointCallSitesIn_SeededSelfTest`
- `internal/planglyph#TestChokepointCallSites_PinnedToTheirGuardedFunctions`
- `internal/planglyph#TestGlyphChain_ResignScenarios`

**Uses:**
- `plan:internal/planglyph#writeLanguagePlan`
- `internal/planglyph#resignCard`
- `internal/planglyph#createCard`
- `internal/planglyph#renameCard`
- `internal/planglyph#renameMechanic`
- `internal/planglyph#deleteCard`

**Intent:** Accept and check a C# re-sign arrow (D7).

`nameDeclarations` owns the one `quarry.Name` call of the package with its length guard, the `ErrQuarryUnavailable`-wrapped error `CanonicalizeHandles` returns today; `CanonicalizeHandles` and `nameResigns` call it, and `chokepointGuardedFuncs` pins `Name` to `nameDeclarations`, with the seeded self-test naming it.

Re-sign arrows move out of `CanonicalizeHandles` into `nameResigns`, which `resolvePass` calls before the plan's batched resolve.
For every Edit arrow on a pending card whose target `planparser.ParseRef` parses to a member glyph, it builds the declaration from that parse: `Unit` the glyph's unit, `Lang` its language, `Decl` the arrow's head, and for a C# target `Owner` `csharpOwner(g, head)`.
A `resignName` records the card, the target glyph, the head and the minted id.
- A Go arrow keeps today's rule: an error or an id other than the target is `resign-head-mismatch`.
- A C# arrow is accepted when the minted id equals the target apart from its parameter list: same unit, owner chain and name, and both with parentheses or both without, so a method is never re-signed into a property.
  A naming error, another member, or a minted id that drops or adds the parameter list is `resign-head-mismatch`, whose C# detail, built in `resignHeadMismatch`, names the way forward for its case:
  for a minted id that drops or adds the parameter list, keep the member's parameter list in the head, or change the member's kind through a Delete and a Create;
  for a head minting another member, a Rename pair for a name change, or a head that declares the target member;
  for a naming error, correct the head to one quarry names under the owner chain, quoting quarry's error, and, when quarry's reason is `invalid_container` and the head writes an explicit-interface qualifier whose spelling differs from its canonical spelling through `glyph.CanonicalCSharpType`, write the qualifier in canonical spelling, quoting the canonical qualifier `glyph.CanonicalCSharpType` gives.
Each accepted C# minted id that differs from its target joins `resolvePass`'s batched resolve target set; `not_found` is its expected answer and raises nothing, since no card names it and `statusFindings` attributes findings only to referencing cards.
An explicit interface member's arrow, such as one on `Glyphchain.Sharp#Renderer.IProvider<int>.Value`, mints its own target id, since `csharpOwner` leaves the qualifier to the head.

`resignCollisionFindings` runs in `resolvePass` after the resolve and after `CanonicalizeHandles`, over the current plan, and reports the new blocking `resign-overload-collision` for an accepted C# arrow whose minted id differs from its target and:
- resolves found, unless the same or an earlier card deletes that id, names it as a Rename old side, or re-signs it away; the detail names the overload and the way forward: change the head to a free signature, or remove or re-sign that overload on an earlier card;
- equals the id a Create handle or Rename new handle elsewhere in the plan binds to (the handle's body once canonical), unless a card between the two declarations deletes, renames away or re-signs away the member the earlier one declares; the detail names that handle's card and the way forward: change the head to a free signature, or drop one of the two declarations;
- equals the minted id of another arrow whose target is a different glyph, with the same exception; the detail names the other arrow's card and the way forward: change one head to a free signature.
Two arrows on the same target never collide with each other.

What it lets through: a later card names a C# member re-signed by an earlier card by its old glyph, which resolves at the plan gate; card 12 rewrites it once the re-sign is recorded.

A collision against another declaration is attributed to the later of the two cards, and its detail names the earlier one.

`handle.go`'s file header, which says `quarry.Name` is called only on `CanonicalizeHandles`' path with `resolvePass` its only caller, is reworded to name `nameDeclarations` as the one site and its callers.
`resolvePass`'s doc comment states the new order: re-sign naming, the batched resolve, handle canonicalization, the collision check, then the status and Create passes.
`chokepoint_enforcement_test.go`'s file header and `TestChokepointCallSites_PinnedToTheirGuardedFunctions`' doc and failure message name `nameDeclarations` where they name `CanonicalizeHandles` as the `quarry.Name` site.

Tests: `TestGlyphChain_ResignScenarios` is the re-sign table, and it gains a `language` field, defaulting to `go`, and a `detail` field, the substrings every reported finding's detail must hold, checked beside the existing finding keys.
Its rows are written as a list of cards (`cards`), so a scenario can put a delete, a Create or a second arrow on another card; the Go rows keep their verdicts as one-card lists.
Its new `csharp` rows run over the fixture:
- `Glyphchain.Sharp#Renderer.Draw(int)` -> `public void Draw(long count)` validates clean, with no `resign-interface-method`;
- `Glyphchain.Sharp#Renderer.Draw(int)` -> `public int Draw` drops the parameter list, and `Glyphchain.Sharp#Renderer.Title` -> `public string Title()` adds one: each is `resign-head-mismatch` whose detail names "keep the member's parameter list";
- `Glyphchain.Sharp#Renderer.Draw(int)` -> `public void Other(int count)` names another member, a `resign-head-mismatch` whose detail names "Rename pair", and `Glyphchain.Sharp#Renderer.Draw(int)` -> `public void Draw(` fails to name, a `resign-head-mismatch` whose detail names "correct the head";
- `Glyphchain.Sharp#Renderer.Draw(int)` -> `public void Draw(string text)` is `resign-overload-collision` whose detail names `Glyphchain.Sharp#Renderer.Draw(string)` and "free signature"; it is clean when an earlier card deletes `Glyphchain.Sharp#Renderer.Draw(string)`, when the same card re-signs `Draw(string)` away to `public void Draw(char c)`, and when an earlier card renames it away to the draft `plan:Glyphchain.Sharp#Renderer.Write`;
- card 1 declaring a Create handle with head `public void Draw(long count)` and card 2 the arrow `Glyphchain.Sharp#Renderer.Draw(int)` -> `public void Draw(long count)` is `resign-overload-collision` on card 2 whose detail names card 1;
- card 1 renaming `Glyphchain.Sharp#Renderer.Draw(string)` to the draft `plan:Glyphchain.Sharp#Renderer.Fill`, which binds to `Fill(string)`, and card 2 re-signing `Glyphchain.Sharp#Renderer.Fill(int)` -> `public void Fill(string text)` is `resign-overload-collision` on card 2 whose detail names card 1, its minted id equal to the Rename new handle's;
- card 1's arrow on `Draw(int)` and card 2's on `Draw(string)`, both minting `Draw(long)`, are `resign-overload-collision` on card 2 whose detail names card 1, while the same arrow on `Draw(int)` written on two cards is clean;
- `Glyphchain.Sharp#Renderer.IProvider<int>.Value` -> `int IProvider<int>.Value { get; }` validates clean;
- `Glyphchain.Sharp#Renderer.IPair<string,int>.First` -> `int IPair<string, int>.First { get; }`, its qualifier written with a space quarry's spelling drops, is `resign-head-mismatch` whose detail names `invalid_container` and the canonical qualifier `IPair<string,int>.`.

**ImpactSummary:** re-sign naming moves from `CanonicalizeHandles` into a pass that runs before the resolve, behind the package's one guarded `quarry.Name` site; Go arrows keep their verdict, and C# arrows gain two blocking checks.
