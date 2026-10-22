# Card 10 — csharp-rename-derivation

**Create:**
- `internal/planglyph#renameCSharpDecl`
- `internal/planglyph#TestRenameCSharpDecl`

**Edit:**
- `internal/planglyph#renameDeclSource`
- `internal/planglyph#isInterfaceMethod`
- `internal/planglyph#TestGlyphChain_CSharp`

**Uses:**
- `internal/planglyph#declaredNameIndex`
- `internal/planglyph#csharpOwner`
- `internal/planglyph#writeLanguagePlan`
- `internal/planglyph#renameCard`
- `internal/planglyph#renameMechanic`
- `internal/planglyph#readCardFile`

**Intent:** Derive a C# `Rename`'s new declaration (D7).

In `renameDeclSource`, when the resolved old symbol's `Glyph.Lang` is `glyph.CSharp`:
- the new identifier is the `Name` of `planparser.ParseRef` over the new handle's body, with its generic arity suffix stripped (`Crate<>` gives `Crate`), never `planparser.HandleIdentifier`'s last-dot split; a body `ParseRef` rejects is `rename-old-unresolved` naming the new side and the way forward: spell the new handle `plan:<namespace>#<owner.member>` in the C# alphabet with canonical parameter types;
- the head is `renameCSharpDecl(sym, identifier)`;
- the declaration is `quarry.Declaration{Lang: glyph.CSharp, Unit: sym.Glyph.Unit, Owner: csharpOwner(sym.Glyph, head), Decl: head}`.
The var and const head synthesis and the `func (Owner)` receiver synthesis stay Go-only: they run only for a Go symbol.

`renameCSharpDecl`:
- for a `field`, `const` or `event` symbol it derives a one-declarator head, since quarry gives every declarator of one declaration the whole declaration as its signature and a substitution in `public int a, b;` would declare two symbols, which `quarry.Name` refuses with `several_declarations`.
  The declarator name is the last identifier at nesting depth zero, outside attribute lists and outside every `()`, `[]` and `<>` pair, before the first depth-zero `,`, `=` or `;`, or before the end of the signature.
  The head is the signature cut before that name, followed by the new identifier, with ` = default` appended for a const.
  So `private Dictionary<string, int> map;` renamed to `renamed` gives `private Dictionary<string, int> renamed`, `private (int, int) pair;` keeps its tuple type, `public const int A = 1, B = 2;` renamed from `A` to `C` gives `public const int C = default`, and an event with accessors, whose signature quarry cuts before them, gives an event field head.
- for every other kind it replaces the declared name token `declaredNameIndex` finds with the new identifier, so `class Box<T>` gives `class Crate<T>` and `public Color Color { get; }` renames its second `Color`.
  Bound: a head whose own type is a generic of the member's name, such as `Color<int> Color { get; }`, has the type's token replaced.
- it calls `declaredNameIndex` with the resolved symbol's own name and never takes its fallback: it keeps the index only when the token there is that name without its generic arity suffix, so a name not found as a whole identifier reports false.
  It reports false when no declared name is found, as for an operator, an indexer, a finalizer (`~Renderer`) or a static constructor (`static Renderer`), which no identifier names; `renameDeclSource` turns that into `rename-old-unresolved` whose detail names the way forward: drop the pair, since such a member is renamed with its type or not at all.
A type's constructors follow the type through the hub's rename tool, with no pair of their own.

`isInterfaceMethod` answers false for every symbol whose `Glyph.Lang` is `glyph.CSharp`, so `resign-interface-method` and the receiver synthesis stay Go-only.

Tests:
- `TestRenameCSharpDecl` pins the exact derived heads the decision record names, which the glyph-chain rows see only through the minted id, over hand-built `quarry.Symbol` values: const `public const int A = 1, B = 2;` from `A` to `C`, giving exactly `public const int C = default`; field `private Dictionary<string, int> map;` to `renamed`, giving exactly `private Dictionary<string, int> renamed`; tuple field `private (int, int) pair;` to `twin`, giving exactly `private (int, int) twin`; property `Color<int> Color { get; }` to `Hue`, giving the bound's `Hue<int> Color { get; }`; an operator signature `public static Box operator +(Box a, Box b)`, which reports false; and a finalizer signature `~Renderer()`, which reports false.
- `TestGlyphChain_CSharp` gains Rename rows over the fixture, each a `csharp` plan with `renameCard` and `renameMechanic`, asserting with `readCardFile` the canonical new handle written into the card file and both gates clean: `Glyphchain.Sharp#Box<>` to `plan:Glyphchain.Sharp#Crate<>`; `Glyphchain.Sharp#Renderer.Draw(int)` with the draft `plan:Glyphchain.Sharp#Renderer.Paint`, canonicalized to `plan:Glyphchain.Sharp#Renderer.Paint(int)`, since quarry mints a method's id with its parameter list, which also shows its head carries no `func` receiver; `Glyphchain.Sharp#Renderer.A` to `plan:Glyphchain.Sharp#Renderer.C`; `Glyphchain.Sharp#Renderer.Color` to `plan:Glyphchain.Sharp#Renderer.Hue`; `Glyphchain.Sharp#Renderer.map` to `plan:Glyphchain.Sharp#Renderer.renamed`; and the accessor event `Glyphchain.Sharp#Renderer.Moved` to `plan:Glyphchain.Sharp#Renderer.Shifted`, whose event field head mints the renamed event's id.
  A further row renames `Glyphchain.Sharp#Renderer.Draw(int)` to the draft `plan:Glyphchain.Sharp#Renderer.Bad(Int32)`, which `ParseRef` rejects: one `rename-old-unresolved` whose detail names the C# handle spelling.
  A `csharp` plan's symbol Rename needs a C# entry in the hub's `rename_tools`; `rename-tool-missing` belongs to webster and is not reached by these gate rows.

**ImpactSummary:** `renameDeclSource` gains a C# branch and `isInterfaceMethod` a C# guard; every Go symbol takes its existing path, so the Go glyph-chain rows keep their findings.
