# Card 1 — shed-retired-rows

**Edit:**
- `internal/shedbuild#Recipe`
- `internal/shedbuild#Parse`
- `internal/shedbuild#ShedPaths`
- `internal/shedbuild#NewShedFrom`
- `internal/shedbuild/doc.go`
- `internal/shedengine#Shed`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine/doc.go`
- `internal/shedbuild#TestParse_ValueAndErrorCases`
- `internal/shedbuild#TestNewShed_FieldCompleteAssembly`

**Create:**
- `plan:internal/shedengine#TestStep_RetiredRowMovesAndPauses` -> `func TestStep_RetiredRowMovesAndPauses(t *testing.T)`

**Uses:**
- `internal/shedengine#TestStep_UnknownCurrentProducerNamesGoto`
- `internal/shedengine#Shed.persistTransient`
- `internal/shedengine#findProducer`

**Intent:** Give shed's shared machinery a recipe-declared map of retired rows (D13), so a run whose status names a row a recipe dropped moves once onto the row the recipe names instead of failing the lookup.

- `shedbuild.Recipe` gains `Retired map[string]string` with the yaml key `retired`: each key a row name the recipe no longer has, each value the row a run on it moves to. `Parse`, the sole recipe parser, refuses a retired key that is also a row of the recipe and a target that names no row, each naming the key and the row (`PATTERN-sole-parsers`). A recipe with no `retired` key parses as today.
- `shedengine.Shed` gains `Retired map[string]string`, `RetiredWayForward string` (the told resume command a move's pause reason and a failed move's error name; empty falls back to `runVerb("run", s.RunID)`) and `RetiredMove func(from, to string, resumed bool) error` (the recipe wiring's side effects of a move; nil does nothing). `shedbuild.ShedPaths` gains `RetiredMove` and `RetiredWayForward`, and `NewShedFrom` copies them and the recipe's `Retired` onto the Shed.
- `Shed.stepLocked`, step 2: when `findProducer` misses and `s.Retired` maps the current row to a target, it calls `RetiredMove(from, target, resumed)` first, where `resumed` is true for a status that is not running (a halted run being stepped again), then persists the status naming the target, state paused, with the reason `the row <from> is retired from the recipe, so the run moved to <target> and paused; way forward: <RetiredWayForward>`, and returns that paused StepResult. A `RetiredMove` error is returned as `shedengine: move the run off the retired row <from>: <error>; way forward: fix the cause it names, then <RetiredWayForward>`, and nothing is persisted, so the next step retries the move; the seam is idempotent (`PATTERN-refusal-way-forward`). The persist runs under the status lock as every persist does. A current row that is a row of the recipe never consults `Retired`, so a run at Finalize or past it is untouched. A current row that is neither a row nor a retired key keeps today's refusal naming goto, unchanged.
- The paused stop is handed back by the driver's existing paused row (`decideDrive`'s "the run is paused: hold and hand back"), never to the tackler or arbiter, so no driver change is needed.
- Bound: only a status naming a key of the recipe's own `retired` map moves; the map is read from the recipe the stepping binary embeds, so an old-binary process keeps running old rows until it stops.

Tests (tier 0): `TestParse_ValueAndErrorCases` gains rows for a `retired` map that parses, a retired key that is also a row, and a target naming no row. `TestNewShed_FieldCompleteAssembly` covers the two new ShedPaths fields and `Retired`. The new `TestStep_RetiredRowMovesAndPauses` builds a Shed whose status names a retired row and asserts: the status names the target, is paused with the reason naming the move and the told way forward, `RetiredMove` was called once with `(from, to, resumed)` where `resumed` is true for a blocked status and false for a running one; a `RetiredMove` error leaves the status file unchanged and returns an error naming the cause and the told way forward; a status at a row the recipe has, with `Retired` set, calls no `RetiredMove` and steps that row. `TestStep_UnknownCurrentProducerNamesGoto` keeps passing unchanged for a row neither present nor retired.

**ImpactSummary:** shedbuild's parser and builder and shedengine's step lookup gain an optional retired map and seam; every recipe without `retired` and every caller leaving the new ShedPaths fields zero behaves as today.

**Commit:** 1: Shed moves a run off a retired recipe row and pauses it
