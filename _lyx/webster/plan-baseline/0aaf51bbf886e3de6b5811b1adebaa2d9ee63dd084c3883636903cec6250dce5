# Card 9 — csharp-create-handles

**Create:**
- `plan:internal/planglyph#declaredNameIndex` -> `func declaredNameIndex(head, name string) (int, bool)`
- `plan:internal/planglyph#csharpOwner` -> `func csharpOwner(g glyph.Glyph, head string) string`

**Edit:**
- `internal/planglyph#CanonicalizeHandles`
- `internal/planglyph#TestGlyphChain_CSharp`

**Uses:**
- `internal/planglyph#writeLanguagePlan`
- `internal/planglyph#createCard`
- `internal/planglyph#readCardFile`

**Intent:** Bind a C# `plan:` handle (D7, D1).

A C# handle is `plan:<namespace>#<owner.member>` with a C# declaration head.
In `CanonicalizeHandles`, when `planparser.ParseRef` over the handle's body returns a C# glyph, its declaration is `quarry.Declaration{Lang: glyph.CSharp, Unit: g.Unit, Owner: csharpOwner(g, head), Decl: head}`: namespace, owner chain and name come from the parse, never from splitting the member at its last `.`, so a parameter list holding a dotted type such as `Paint(System.Text.Rune)` is not cut.
A Go handle keeps the card 3 declaration with no `Owner`.

`declaredNameIndex` returns the byte index of the declared name token in a head.
With a non-empty `name` it is the first occurrence of `name`, without its generic arity suffix (`Box<>` as `Box`), as a whole identifier outside attribute lists (`[...]` at the head's start or after whitespace), that is followed, after optional whitespace, by `(`, `<`, `{`, `=`, `;`, `:`, the word `where` or the end of the head.
With an empty `name`, or when that name is not found, it is the declarator position: the last identifier at nesting depth zero, outside attribute lists and outside every `()`, `[]` and `<>` pair, before the first depth-zero `(`, `{`, `=`, `;`, `=>`, `:`, `,`, the word `where` or the end of the head, so a draft handle whose member name the head does not write still finds the head's own declared name.
A keyword is never an identifier here, so `public`, `void` and `int` are skipped, and `[` is no stop token, so an array type such as `int[] IProvider<int>.Items { get; }` gives `Items`.
So in `public Item.Value Value { get; }` it is the second `Value`, in `class Box<T>` it is `Box`, in `public void Wrap<U>(T item) where U : class` it is `Wrap`, and in `int IProvider<int>.Count { get; }` it is `Count` whatever the draft name.
It reports false when no identifier qualifies, when that identifier is preceded by `~` or by the word `static` with no type between them, as in a finalizer `~Renderer()` or a static constructor `static Renderer()`, and when the head holds the keyword `operator` or `this` at depth zero, as an operator or an indexer does.

`csharpOwner` returns the glyph's owner chain joined with `.` with the member's interface qualifier removed.
The qualifier is the longest trailing run of owner-chain components the head writes, dotted and followed by a dot, immediately before the declared-name position, compared in quarry's canonical C# spelling through `glyph.CanonicalCSharpType`, so `IPair<string, int>.` in a head matches the component `IPair<string,int>`.
The declared-name position is the token `declaredNameIndex(head, g.Name)` finds, or, when it reports false, a depth-zero `this` followed by `[`, so `int IProvider<int>.this[int index] { get; }` for the glyph `Renderer.IProvider<int>.this[int]` gives the owner `Renderer`; `declaredNameIndex` itself keeps reporting false for that head, which card 10's Rename relies on.
A head that does not write the qualifier components there, or in which no declared-name position is found, leaves the chain whole: `void System.IDisposable.Close()` for `Renderer.System.IDisposable.Close()` and `int IProvider<int>.Count { get; }` for `Renderer.IProvider<int>.Count` both give the owner `Renderer`, while `Item.Kind Get()` for `Outer.Item.Get()` gives `Outer.Item`, since `Item.Kind ` is a type, not a qualifier immediately before `Get`.
Quarry refuses an owner component with type arguments and reads the head's qualifier itself, which is why the qualifier must leave the owner.
`csharpOwner` compares in canonical spelling, so the owner it hands quarry is `Renderer`, but quarry `v0.4.0` (`internal/engine/name.go` `checkMintedContainer`) matches the qualifier against the head as written, so a head must spell the qualifier canonically (`IPair<string,int>.`) to bind.
When quarry's reason for a C# Create handle is `invalid_container` and the head writes an explicit-interface qualifier whose spelling differs from its canonical spelling through `glyph.CanonicalCSharpType`, `CanonicalizeHandles` appends the way forward to the `handle-name-failed` detail: write the qualifier in the head in canonical spelling, quoting the canonical qualifier `glyph.CanonicalCSharpType` gives (`IPair<string,int>.` for `IPair<string, int>.` or `IPair< string,int>.`).

Tests: `TestGlyphChain_CSharp` gains Create rows, each a `csharp` plan with one `createCard`, asserting with `readCardFile` the canonical handle written into the card file and both gates clean:
- `plan:Glyphchain.Sharp#Renderer.Paint(System.Text.Rune)` -> `public void Paint(System.Text.Rune glyph)` canonicalizes to itself, its parameter list holding a dotted type;
- `plan:Glyphchain.Sharp#Renderer.IProvider<int>.draft` -> `int IProvider<int>.Count { get; }`, whose draft name the head does not write, binds with the owner `Renderer` and canonicalizes to `plan:Glyphchain.Sharp#Renderer.IProvider<int>.Count`;
- `plan:Glyphchain.Sharp#Renderer.IPair<string,int>.First` -> `int IPair<string, int>.First { get; }`, written with a space quarry's spelling drops, takes the owner `Renderer` and is the one exception to the clean gates: one `handle-name-failed` whose detail carries quarry's `invalid_container` and the clause quoting the canonical qualifier `IPair<string,int>.`;
- `plan:Glyphchain.Sharp#Renderer.System.IDisposable.Close()` -> `void System.IDisposable.Close()`, a two-component qualifier, binds with the owner `Renderer`;
- `plan:Glyphchain.Sharp#Renderer.IProvider<int>.this[int]` -> `int IProvider<int>.this[int index] { get; }`, an explicit-interface indexer, takes the owner `Renderer` and canonicalizes to itself, needing no fixture member since a Create handle is named without resolving;
- `plan:Glyphchain.Sharp#Renderer.Mark()` -> `[Obsolete] public void Mark()` finds `Mark` past the attribute list;
- `plan:Glyphchain.Sharp#Box<>.Wrap<>(T)` -> `public void Wrap<U>(T item) where U : class` finds `Wrap` before its `<`;
- `plan:Glyphchain.Sharp#Outer.Item.draft` -> `public Item.Kind Get()` binds with the owner `Outer.Item` and canonicalizes to `plan:Glyphchain.Sharp#Outer.Item.Get()`.

**ImpactSummary:** `CanonicalizeHandles` gains a C# branch for Create declarations only; Go handles and every Go fixture row keep their declaration.
