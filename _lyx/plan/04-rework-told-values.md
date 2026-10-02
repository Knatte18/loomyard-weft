# Card 4 — rework-told-values

**Edit:**
- `internal/loomshed/prrework.go`
- `internal/loomshed/prrework_test.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_prrework.go`
- `internal/shedrecipe/entries_prrework_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_rework_integration_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomrecipe/shape_test.go`
- `internal/shedbuild/fixture_test.go`

**Create:**
- `plan:internal/loomshed#ReworkTold` -> `type ReworkTold struct`

**Intent:**
Move the decision of what the rework session is told out of the wiring and into the PR-Rework producer, so the next card can tell the session where it archived the retired generation.
Behaviour is unchanged by this card: the session is still told the number one past the highest card committed at HEAD.

- `ReworkTold` carries the values the producer tells its session; this card gives it `FirstCard int`.
- `NewPRRework` takes a session factory, `func(ReworkTold) shedengine.ShedProducer`, in place of the ready-built inner producer.
  `prRework.Call` computes `FirstCard` with `NextReworkCardNumber` over `deps.PlanDir` and `deps.ReadCommitted` before the session runs, builds the session from the factory, and calls it.
- `Env.ReworkSpec` becomes `func(loomshed.ReworkTold) (shuttleengine.Spec, error)`.
  `prReworkEntry` builds the factory: each call returns `shedadapters.NewSingleLLMProducerGated` over a spec source closing over the told value, with the same gate, shuttle and clock as today, so attach-if-live and the gate behave as before.
- `wire()`'s `ReworkSpec` closure stops calling `NextReworkCardNumber` itself and passes `told.FirstCard` to `loomengine.ReworkSpec`.

Tests: `prrework_test.go` asserts the factory receives the next card number computed from the committed plan; `entries_prrework_test.go` asserts the built producer evaluates `Env.ReworkSpec` with the told value; the loomrecipe and shedbuild fixtures and the shape test's `NewPRRework` call follow the new signatures; the rework integration test calls `ReworkSpec` with a told value.

**ImpactSummary:** PR-Rework's constructor and `Env.ReworkSpec` change signature; their callers are the PRRework registry entry, `wire()`, and test fixtures in loomshed, shedrecipe, shedbuild, loomrecipe and loomcli.
