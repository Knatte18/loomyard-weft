# Card 2 — glyphchain-delta-legs

**Create:**
- `internal/planglyph/glyphchain_integration_test.go`

**Edit:**
- `internal/planglyph/glyphchain_test.go`

**Uses:**
- `internal/planglyph#newDeltaFixtureRepo`
- `internal/planglyph#Delta`
- `internal/planglyph#BindHandles`
- `internal/planglyph#DoneChecks`

**Intent:**
Add the integration half of the D1 test: each shape row's Create, Rename and Delete legs carried past the plan gate through a real commit's delta, handle binding, the on-disk rewrite and the done-checks, plus the `rework-generations` scenario row.

`glyphchain_test.go`'s `shapeRows` gains each leg's code change: the text a Create leg appends or inserts (the interface method into `Iface`'s body, the multi-name and block members into their blocks), the replacements a Rename leg applies (receivers included, so a renamed generic type keeps its method compiling), and the text a Delete leg removes (for the `shapes#Generic` row, the type and its method together).

`glyphchain_integration_test.go` (`//go:build integration`; it spawns git through `Delta`) runs, per row and leg, in its own repository from `newDeltaFixtureRepo` with the fixture copied in and committed as the base:
- Create: validate the leg's plan as card 1 does, apply the code change and commit, take `Delta(base, head)`, and call `BindHandles` for the creating card.
  It reports no finding, the creating card's declaration bullet collapses to `` - `<glyph>` ``, the referencing card carries the same glyph, and `DoneChecks` over the creating card reports nothing.
- Rename: the same over the Rename card; the pair's to-side binds to the renamed glyph and `DoneChecks` reports nothing.
- Delete: apply the removal and `DoneChecks` over the Delete card reports nothing.

The `rework-generations` row is three cards: card 1 creates `plan:shapes#Told` -> `type Told struct` and edits `shapes/shapes.go`, card 2 edits `shapes/shapes.go` and `plan:shapes#Told`, card 3 edits `shapes/shapes.go`.
It validates clean at the plan gate; after card 1's code is committed and bound, `ValidateDispatch` with card 1 completed reports the blocking `containment-file-overlap` it reports today, the verdict change card 5 removes.

**ImpactSummary:** Test-only: extends the untagged glyph-chain table with code changes and adds one integration test file; no production code changes.
