# Card 10 — darn-producer-and-entry

**Create:**
- `plan:internal/loomshed#NameDarn` -> `const NameDarn = "Darn"`
- `plan:internal/loomshed#darnResumeWayForward` -> `const darnResumeWayForward = "way forward: run \"lyx loom resume\" in the task worktree, which re-spawns Darn with a fresh verify budget"`
- `plan:internal/loomshed#DarnTold` -> `type DarnTold struct`
- `plan:internal/loomshed#DarnOutcome` -> `type DarnOutcome struct`
- `plan:internal/loomshed#DarnDeps` -> `type DarnDeps struct`
- `plan:internal/loomshed#darnWrite` -> `type darnWrite struct`
- `plan:internal/loomshed#NewDarnWrite` -> `func NewDarnWrite(name string, session func(DarnTold) shedengine.ShedProducer, deps DarnDeps) shedengine.ShedProducer`
- `plan:internal/loomshed#darnWrite.Call` -> `func (p *darnWrite) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/loomshed#renderPriorWork` -> `func renderPriorWork(latest DarnOutcome, found bool, failureRecord string) string`
- `internal/loomshed/darnwrite_test.go`
- `plan:internal/shedrecipe#darnWriteEntry` -> `func darnWriteEntry(name string, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe/entries_darnwrite_test.go`
- `contracts/recipes/darn-recipe.yaml`
- `plan:contracts/recipes#DarnRecipe` -> `var DarnRecipe []byte`

**Edit:**
- `internal/loomshed/doc.go`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#registry`
- `internal/shedrecipe/registry_test.go`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe/recipe_test.go`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#intentionallyNil`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D3, D4 and D5: the `Darn` row's producer, its registry entry and the recipe that names it.
`DarnTold` carries the two token texts the producer decides, `RejectionFindings` and `PriorWork`, each empty when absent; `DarnOutcome` is the latest `Darn` history entry's outcome and the halt reason recorded with it, empty when the run did not halt there.
`DarnDeps` carries the told seams: `ReadRejection` and `ClearRejection` in `PRReworkDeps`'s shapes, `Commit`, which commits the landing directory, `LatestOutcome func() (DarnOutcome, bool, error)` over the run's status file, and `PublishFailure func() (string, bool, error)`, the Publish failure record's path when one is present.
`darnWrite.Call`, after the entry check and a named-seam check like `prRework.checkSeams`, reads the pending rejection (a read error is Stuck naming `lyx loom reject`, as `prRework` does), reads the latest outcome and the failure record, renders the prior-work token through `renderPriorWork` (a Go-written note that the change is already under way on the task branch, naming the latest outcome's reason and the record's path, or empty when neither exists), builds the session with `session(told)` and calls it.
On a non-empty output pointer (Done or a gate-failed Stuck) it runs `Commit`, a failure being a returned error in `discussionWrite`'s wording;
on Done of a spawn that rendered a rejection it then runs `ClearRejection`, a failure being a returned error;
on a gate-failed Stuck it keeps the record and returns the pointer with `darnResumeWayForward` appended to its `Reason`, keeping the gate's findings and log path.
`darnWriteEntry` resolves the row's `gates` through `resolveGateSpec` with the row name, rejects other keys, requires `Env.DarnSpec`, `Env.Shuttle` and every `Env.Darn` seam, and returns `NewDarnWrite` over a session factory that wraps `shedadapters.NewSingleLLMProducerGated` around `env.DarnSpec(told)`, as `prReworkEntry` does;
`Env` gains `DarnSpec func(loomshed.DarnTold) (shuttleengine.Spec, error)` and `Darn loomshed.DarnDeps`, and the registry gains `"DarnWrite": darnWriteEntry`.
`darn-recipe.yaml` is the D3 table: entry `Preflight`, terminal `Friction-Reflect`, rows `Preflight` (engine Preflight, on_done Darn), `Darn` (engine DarnWrite, segment PR-Review, on_done Publish, no on_stuck, gates `verify` then `description` each with the literal `attempts: 3`, with a comment that the builder stamps the verify budget from `darn.yaml` at every build), `Publish` (on_done PR-Gate), `PR-Gate` (engine PRGate, segment PR-Review, max_bounces 5, on_stuck Darn, on_done Finalize), `Finalize` (on_done Friction-Reflect) and `Friction-Reflect` (on_done empty), with a header comment stating the rejection edge's bound: each traversal is one operator rejection, and a `Darn` Done ends the gate's bounce episode;
`recipes.go` embeds it as `DarnRecipe`, and `RecipeEngines` unions the engine sets of both recipes so the cross-consumer coverage guard reaches `DarnWrite`.
`FullEnv` fills the two new seams, and `intentionallyNil` lists them as darn-only seams wire fills for a darn seed.
Tests: `darnwrite_test.go` drives the producer with a fake session over every outcome path, pinning the token texts, the commit and clear order, the kept record on a gate-failed Stuck, the appended way forward, and the seam-check errors;
`entries_darnwrite_test.go` pins the construction refusals and the happy path like the describe entry's test;
the registry names test and the loom engine-set test gain `DarnWrite`.
The refusal spec gains the row `Darn verify gate exhausted` with the resume way forward, reached by the producer test.
The loomshed doc gains the row's paragraph.

**ImpactSummary:** One new producer, one registry row and two `Env` seams; the recipe is embedded but not yet built, so no run's behavior changes until the builder card.
