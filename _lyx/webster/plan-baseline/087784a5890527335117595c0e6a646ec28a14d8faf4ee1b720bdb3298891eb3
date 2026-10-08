# Card 1 — glyphchain-plan-gate

**Create:**
- `internal/planglyph/testdata/glyphchain/go.mod`
- `internal/planglyph/testdata/glyphchain/shapes/shapes.go`
- `internal/planglyph/testdata/glyphchain/shapes/shapes_test.go`
- `internal/planglyph/testdata/glyphchain/shapes/external_test.go`
- `internal/planglyph/testdata/glyphchain/shapes/tagged.go`
- `internal/planglyph/testdata/glyphchain/dirname/clause.go`
- `internal/planglyph/testdata/glyphchain/cmd/tool/main.go`
- `internal/planglyph/glyphchain_test.go`

**Uses:**
- `internal/testkit/plankit#Render`
- `internal/planglyph#readCardFile`

**Intent:**
Build the committed glyph-chain fixture and the untagged half of the D1 test: every member shape class real plans name, driven through parse, the resolve status policy, the Create inversion, handle canonicalization and the on-disk rewrite, asserting the chain as it stands today.

The fixture is a Go module under `internal/planglyph/testdata/glyphchain` (`go.mod`: `module example.com/glyphchain`).
The one fixture member that references another is `GenericMethod`, whose receiver names `Generic` outside the type's resolved span; every leg that deletes or re-signs `shapes#Generic` therefore also lists `shapes#Generic.GenericMethod` on the same card, so card 11's caller coverage, which counts that receiver as a reference, changes no shape row's outcome.
- `shapes/shapes.go`, `package shapes`: `func Func() int`, `type Plain struct{}`, `type Struct struct { Field int }` with `func (s Struct) ValueMethod() int` and `func (s *Struct) PointerMethod()`, `type Iface interface { IfaceMethod() int }`, `type Generic[E any] struct{ value E }` with `func (g *Generic[E]) GenericMethod() E`, `var Var = 1`, `const Const = 2`, a var block holding `BlockVar = 3` and the multi-name spec `PairA, PairB = 4, 5`, and a const block holding `BlockConst = iota` and the implicit `NextConst`.
- `shapes/shapes_test.go`, `package shapes`: `func testHelper() int`.
- `shapes/external_test.go`, `package shapes_test`: `func TestExternal(t *testing.T)`.
- `shapes/tagged.go`, `//go:build integration`, `package shapes`: `func Tagged()`.
- `dirname/clause.go`, `package clausename`: `func DirDiffers()`.
- `cmd/tool/main.go`, `package main`: `func main()` and `func run()`, with no call between them.

`glyphchain_test.go` (package `planglyph`, untagged: these legs read files and spawn no process) holds:
- a fixture copier that copies `testdata/glyphchain` into a fresh `t.TempDir()` with `os.CopyFS`;
- a plan writer like `writePlanFixture` that can also write a `## Rename mechanic` section into the overview (through `plankit.Render`'s `Sections`);
- `shapeRows`, one row per shape class: the member glyph, the expected Edit-leg findings, a Create leg (draft handle, declaration head, canonical handle), a Rename leg (Old glyph, draft to-side handle, canonical handle), a Delete leg, and an optional gap comment;
- `TestGlyphChain_PlanGate`, a parallel subtest per row and leg, each against its own fixture copy and plan directory, comparing the exact set of `(Check, Card, Severity)` findings `ValidateFormat` returns.

Rows and glyphs: `shapes#Func`, `shapes#Plain`, `shapes#Var`, `shapes#Const`, `shapes#Struct.ValueMethod`, `shapes#Struct.PointerMethod`, `shapes#Struct.Field`, `shapes#Iface.IfaceMethod`, `shapes#Generic`, `shapes#Generic.GenericMethod`, `shapes#testHelper`, `shapes_test#TestExternal`, `shapes#Tagged`, `dirname#DirDiffers`, `cmd/tool#run`, `shapes#BlockVar`, `shapes#NextConst`, `shapes#PairB`.

Legs:
- Edit: one card editing the member validates clean.
  The `shapes#Struct.Field` row asserts quarry's current answer, a blocking `glyph-not-found`, with a comment naming the quarry gap: quarry indexes no struct fields, so a field glyph never resolves.
  It has no other leg.
- Create: card 1 creates a draft handle whose member is spelled differently from its head (`plan:shapes#draft` -> `func NewFunc() int`, and for a method `plan:shapes#Struct.draft` -> `func (s *Struct) NewPointerMethod()`), and card 2 references it under `**Edit:**`.
  It validates clean, both card files then carry the canonical handle, and a second `ValidateFormat` leaves both files byte-identical.
  The interface-method row's head is `func (i Iface) NewIfaceMethod() int`; the multi-name row's head is `var NewPairB int`.
- Rename: one card renames the member to a draft handle spelled in the wrong unit (`plan:dirname#FuncRenamed`, and `plan:shapes#DirDiffersRenamed` for the `dirname` row); it validates clean and the card file carries the handle in the Old glyph's unit.
  The interface-method and multi-name rows carry no Rename leg yet: their renames fail today, and cards 3 and 4 add those legs with their fixes.
- Delete: one card deleting the member validates clean.
  The `shapes#Generic` row's card deletes `shapes#Generic` and `shapes#Generic.GenericMethod` together, since the method cannot outlive its receiver type.

Cross-card rows in the same file, asserting today's refusals that card 5 flips:
- a member on one card and its file on another: blocking `containment-file-overlap`;
- a member, or a file, on one card and its package self glyph `shapes#` on another: blocking `containment-unit-overlap`;
- the same file on two cards: clean;
- a Create handle no other card references: blocking `handle-unreferenced`, which card 6 flips.
