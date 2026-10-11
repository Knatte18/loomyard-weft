# Card 2 — The code index builds a batch's kick-start pack

**Create:**
- `plan:internal/planindex#PackDeclaration` -> `type PackDeclaration struct`
- `plan:internal/planindex#PackEntry` -> `type PackEntry struct`
- `plan:internal/planindex#PackUnresolved` -> `type PackUnresolved struct`
- `plan:internal/planindex#Pack` -> `type Pack struct`
- `plan:internal/planglyph#KickstartPack` -> `func KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) (planindex.Pack, error)`
- `plan:internal/planglyph#index.KickstartPack` -> `func (index) KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) (planindex.Pack, error)`
- `internal/planglyph/pack_test.go`
- `plan:internal/planglyph#TestKickstartPack` -> `func TestKickstartPack(t *testing.T)`

**Edit:**
- `internal/planindex#Index`
- `internal/planglyph/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/planglyph#Resolve`

**Intent:**
Decisions D4 and D5 call for a pack built from a batch's cards and resolved through the code index.
`internal/websterengine` gets it without linking quarry or the C grammars.
Card 7 calls it from `BeginBatch`.

In `internal/planindex/planindex.go`, declare the pack types with standard-library fields only, so `planindex` keeps its import set:
- `PackDeclaration`: `File string`, `Start int`, `End int`, `Signature string`.
- `PackEntry`: `Glyph string`, `Declarations []PackDeclaration`.
- `PackUnresolved`: `Ref string`, `Reason string`.
- `Pack`: `Resolved []PackEntry`, `Unresolved []PackUnresolved`.
Add `KickstartPack(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) (Pack, error)` to the `Index` interface.
It is an interface method, so it takes no arrow.
Its doc comment states the degraded-pack contract below.
The two implementers of `Index` are `planglyph#index` and test types that embed `planindex.Index`.
The embedded interface gives the test types the method, so they need no change on this card.

`planglyph.KickstartPack` lives in a new `internal/planglyph/pack.go`, and `index.KickstartPack` forwards to it.

Selecting refs:
- Walk `cards` in order.
- For each card, take every ref under a target label in body order:
  - the `Refs` of an `Edit`, `Delete` or `Move` group
  - the `Old` side of each `Rename` pair
- Then take the card's `Uses`.
- Keep each spelling's first occurrence only.
- Under `language: none` (`plan.GlyphLanguage` false), return an empty `Pack` and no error, with no quarry call.

Sorting the refs:
- Every `plan:`-shaped ref on the cards, under any label including `Create` and a Rename `New` side, goes to `Unresolved`.
  `planparser.IsHandleRef` decides this.
  Its fixed reason is that it names a member a card creates and does not resolve yet.
- Every other ref goes through `glyph.Parse` (`PATTERN-glyph-conversion-chokepoint`).
  A ref that fails to parse is a plain path and is left out.
  So is a self glyph.
- Only member glyphs are resolved, all in one `planglyph.Resolve` call against `worktreeRoot`.

Building entries from the answer:
- A `found` or `multipart` answer becomes a `PackEntry`.
  It carries one `PackDeclaration` per quarry symbol, with `File`, `Start`, `End` and `Signature` copied verbatim from the answer.
  A path is never derived from a glyph.
- A `not_found` or `ambiguous` answer goes to `Unresolved` with its quarry status as reason.
- A grammar rejection goes to `Unresolved` with its error and reason.
- Entries and unresolved refs keep selection order.

Degraded pack: when the `Resolve` call fails as a whole, return a pack that lists every member glyph under `Unresolved`, with the error text as reason, beside the `plan:` refs, and return the `ErrQuarryUnavailable`-wrapped error.
That way the caller degrades instead of refusing, as `index.Delta` does with its empty delta.

`TestKickstartPack` is a new table test in a new `internal/planglyph/pack_test.go`.
It runs against fixture trees written to `t.TempDir()`, as `internal/planglyph/donecheck_resolve_test.go` does.
It asserts the exact `Pack` for these rows:
- Edit, Delete, Rename-old and Uses member glyphs resolve with file, span and signature.
- A ref repeated across two cards appears once, at its first card's position.
- Self glyphs and plain paths are left out.
- A Create target and a Rename new side are not resolved.
- A `plan:` ref under Create and under Uses is listed unresolved with the fixed reason.
- A `not_found` member glyph is listed unresolved with its status.
- A `language: none` plan returns an empty pack.
- A worktree quarry cannot open returns the degraded pack and an `ErrQuarryUnavailable` error.

Raise the `internal/planglyph` row in `cmd/lyx/testdata/test-budget.yaml` by one.
Add `KickstartPack` to `internal/planglyph/doc.go`'s description of what the package serves through `planindex.Index`.

**ImpactSummary:** the `planindex.Index` interface gains one method, implemented by `planglyph#index` and inherited by every embedding test fake; no existing caller's behavior changes until card 7 calls it.
