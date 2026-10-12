# Card 4 — shedbuild-on-reroute

**Edit:**
- `internal/shedbuild#Row`
- `internal/shedbuild#Build`
- `internal/shedbuild#RoutingOf`
- `internal/shedbuild#ShedPaths`
- `internal/shedbuild#NewShedFrom`
- `internal/shedbuild#TestBuild_RowsCarryRoutingInRecipeOrder`
- `internal/shedbuild#TestRoutingOf_MatchesNewShedRowForRow`
- `internal/shedbuild#TestNewShed_FieldCompleteAssembly`
- `internal/shedbuild#TestParse_ValueAndErrorCases`
- `contracts/specs/shed-recipe-spec.md`

**Intent:**
Carry card 3's reroute edge through the recipe layer.
`Row` gains `OnReroute` under the YAML key `on_reroute`; `Build` and `RoutingOf` copy it onto `ProducerDef.OnReroute`, beside `OnDone` and `OnStuck`.
`ShedPaths` gains `RerouteCapWayForward`, which `NewShedFrom` copies onto the built `Shed` as it copies `MissingStatusWayForward`.
`Parse` needs no new check: `shedengine`'s `validate` already refuses an `on_reroute` naming a row not in the list, which is the acceptance criterion "a recipe validation rejects an `on_reroute` naming a row not in the list"; `TestParse_ValueAndErrorCases` gains a row decoding the key.
The existing assembly tests each gain the field: a row with `on_reroute` builds and projects with its target, and `TestNewShed_FieldCompleteAssembly` asserts the copied clause.
`shed-recipe-spec.md` lists `on_reroute` in the row field set and states that its target may sit in another segment.

**ImpactSummary:** one optional recipe key and one told clause pass through unchanged; recipes without the key build exactly as before.

**Commit:** 4: shedbuild: on_reroute recipe key and the reroute-cap way forward
