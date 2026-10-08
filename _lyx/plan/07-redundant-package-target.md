# Card 7 — redundant-package-target

**Create:**
- `internal/planparser/redundancy.go`

**Edit:**
- `internal/planparser/validate.go`
- `internal/planparser/shape.go`
- `internal/planparser/validate_test.go`
- `internal/planglyph/glyphchain_integration_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planparser#HandleUnit`

**Intent:**
Add D3's pure half: the blocking check `redundant-package-target`, a card whose own targets list a package self glyph beside a member or a file of that same package.

`redundancy.go` declares the check, which `validate` in `validate.go` runs after `checkUsesLaterTarget`, under a glyph-enabled `language:` only; the file's header list gains the ID.
It walks each card's own flat `Targets` (every group, both sides of a Rename pair; never `Uses`), classifying each through a new ledger gate in `shape.go` (glyph and handle kept, path and bare symbol skipped):
- a self glyph whose `UnitPath` carries no file extension is a package container;
- a member glyph belongs to its parsed `Glyph.Unit`;
- a file self glyph belongs to the package whose `UnitPath` is its own `UnitPath`'s directory;
- a `plan:` handle that some Rename pair in the plan claims as its to-side belongs to that pair's Old glyph's unit, the unit `CanonicalizeHandles` writes into it; any other handle belongs to `HandleUnit`'s unit.

One finding per container and contained target, attributed to the card, with `Ref` the contained target.
Its detail names both and the way forward: keep the member glyphs and drop the package self glyph, or keep the package self glyph when the card changes the whole package.
Its verdict is the same before and after record-batch binds a handle, since binding keeps the unit, so it runs wherever `planparser.ValidateFormat` runs, `ValidateDispatch` included.

`validate_test.go` covers the pure cases: a package self glyph beside a member, a file, or a handle of that package is flagged; beside a member of another unit, or with the member on a different card, it is clean; under `language: none` nothing is reported.
`glyphchain_integration_test.go` gains the cross-gate rows:
- card 1 creates `plan:shapes#Fresh` and card 2 lists `shapes#` and that handle: flagged at the plan gate, and again by `ValidateDispatch` once card 1's code is committed and bound;
- card 1 renames `shapes#Func` to `plan:dirname#FuncMoved` and card 2 lists `dirname#` and that handle: clean at the plan gate and at dispatch after binding;
- the same with card 2 listing `shapes#`: flagged at both.

The spec gains the row at the position the check runs, renumbered with every `row N` cross-reference fixed.

**ImpactSummary:** Adds one pure blocking check at every gate that runs planparser.ValidateFormat; plans already clean of same-card package redundancy are unaffected.
