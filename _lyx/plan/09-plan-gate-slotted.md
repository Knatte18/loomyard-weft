# Card 9 — plan-gate-slotted

**Create:**
- `plan:internal/planglyph#NewSlottedIndex` -> `func NewSlottedIndex(matcher planparser.FabricReferenceMatcher, slots *gateslot.Pool) planindex.Index`
- `plan:internal/planglyph#validateFormat` -> `func validateFormat(plan *planparser.Plan, worktreeRoot string, loader typesLoader) ([]Finding, error)`
- `plan:internal/planglyph#validateFormatAfter` -> `func validateFormatAfter(plan *planparser.Plan, worktreeRoot string, done []planparser.Card, loader typesLoader) ([]Finding, error)`

**Edit:**
- `internal/planglyph#NewIndex`
- `internal/planglyph#index`
- `internal/planglyph#index.ValidateFormat`
- `internal/planglyph#index.ValidateRework`
- `internal/planglyph#ValidateFormat`
- `internal/planglyph#ValidateFormatAfter`
- `internal/planglyph#ValidateRework`
- `internal/planglyph#goListLoader`
- `internal/planglyph#goListLoader.load`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/planglyph/glyphchain_integration_test.go`
- `internal/planglyph/doc.go`

**Uses:**
- `plan:internal/hubgeom#GateSlots`
- `internal/websterengine#Geometry`

**Intent:**
Slot the plan gate's `caller-uncovered` type load, the one plan-gate step that runs `go list` with export data (D4), at both its gate site and `lyx loom validate-plan`, which both reach it through the index loom wires.
`goListLoader` gains an optional pool: with one, `load` acquires a slot before `packages.Load` with a `Holder` naming the root and the site `plan gate caller-uncovered`, starts its timeout only once the slot is held, sets `GOFLAGS` to `gateslot.GoFlags("-mod=readonly", <cap>)` in place of the bare `-mod=readonly`, and releases the slot when the load returns; without one it behaves as today.
`index` gains the loader it validates with: `NewIndex` keeps its signature and uses `defaultTypesLoader`, read at construction, and `NewSlottedIndex` builds a `goListLoader` over the told pool, or uses `defaultTypesLoader` when the pool is nil.
`validateFormat` and `validateFormatAfter` are the loader-taking bodies of `ValidateFormat` and `ValidateFormatAfter`, which keep their signatures and pass `defaultTypesLoader`; `ValidateRework` delegates the same way, and `index.ValidateFormat` and `index.ValidateRework` pass the index's own loader, so the gate and the verb still call one function (`PATTERN-gate-self-check-parity`).
`loomCLI.wire` builds its index with `NewSlottedIndex(…, websterGeom.GateSlots)` and `wireLightweight` with `NewSlottedIndex(…, hubgeom.GateSlots(location))`; `wireHub` and `wireStandalone` pass `geom.GateSlots`, nil in standalone.
The untagged tier keeps its failing stand-in loader, so no unit test spawns `go list`; `glyphchain_integration_test.go` gains a row in which a one-slot pool held by the test keeps the slotted load waiting until the test releases it, after which it returns the same findings as the unslotted load.
`doc.go` states the slot step.

**ImpactSummary:** Production plan-gate indexes in loom and webster now slot their type load; test callers of `NewIndex` are unaffected because its signature and loader stay the same.
