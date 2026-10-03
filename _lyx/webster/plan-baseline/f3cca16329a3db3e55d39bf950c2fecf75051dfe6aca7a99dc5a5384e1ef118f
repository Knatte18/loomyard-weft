# Card 3 — reflect-under-step

**Edit:**
- `internal/loomcli/run.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/friction_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomshed/interruptpolicy.go`
- `contracts/recipes/loom-recipe.yaml`

**Uses:**
- `internal/loomshed#NewFrictionReflect`
- `internal/loomshed#NameFrictionReflect`
- `internal/testkit/shedfake#Shuttle`
- `internal/testkit/stencilkit#SeedInto`
- `internal/testkit/locationkit#Location`

**Intent:**
The Friction-Reflect row reflects under `step` as under `run` (D1), since every driven run uses `step` and the gate meant none ever reflected.

`reflectFrictionRow` drops the `armedVerb == "run"` condition: with a non-empty friction directory it calls `reflectFriction(true)`, otherwise it reports `StatusSkipped`; it still records the status on `rowFrictionStatus`, and the row still always returns Done, so done persists only after the reflection returns.
`loomCLI.armedVerb`'s only production reader is that condition, and its only other reader is `wiring_test.go`'s `TestArmAt_RecordsTheArmingVerb`, so the field, its assignment in `armAt`, every comment citing it (`armAt`'s doc, the wiring comment on `ReflectFriction`, the `loomCLI` field doc) and that test go: the property it pins no longer exists.

`loomCLI` gains a `reflectionShuttle frictionengine.Shuttle` field, assigned the constructed shuttle runner in `wire` beside `c.runner`, and `reflectFriction` passes it as `Deps.Shuttle`.
It is the seam the tests below and card 6's scenario fake the reflection agent through; on the lightweight wiring path it stays nil, which `Reflect`'s Deps validation reports as `failed`, as today.

What this lets through, and the bound: the reflection agent now files public issues during a driven step with no operator in the loop, per the operator's decision; the step blocks for at most `friction_timeout_min`, and filing goes only through `lyx selfreport create` to the hardcoded `Knatte18/loomyard` repo.

`InterruptPolicies`' doc replaces the Friction-Reflect premise "armed for step the row spawns nothing" with the attach premise from card 1: `frictionengine.Reflect` probes shuttle's attach seam for a live reflection agent and archives a finished reflection's notes instead of spawning, so re-invoking an interrupted step never double-files; the table value stays `reinvoke`.
The loom recipe's Friction-Reflect comment and the `lyx loom` command's long help stop saying the reflection is run-only or skipped under step.

`friction_test.go`:

- `newRelativeFrictionCLI` drops its verb parameter;
- `TestReflectFrictionRow_SkipsWhenArmedForStep` is removed, since there is no armed verb left to gate on;
- `TestReflectFrictionRow_ReflectsWhenArmedForRun` becomes `TestReflectFrictionRow_ReflectsWhenTierTwoOn`, its doc naming Tier 2 on rather than an armed verb;
- a new test builds a receiver over a temporary location with seeded stencils, a non-empty `friction` model-spec and a `shedfake.Shuttle` answering `OutcomeDone`, writes one note, runs `reflectFrictionRow`, and asserts one spawn whose prompt names the note, the note archived, and `rowFrictionStatus` reported as `reflected`;
- the Tier 2 off test keeps asserting `skipped` with no spawn.

**ImpactSummary:** confined to `internal/loomcli`'s friction closure, one dead receiver field removed with the one `wiring_test.go` test that pinned it, one new seam field, and comments in `loomshed` and the loom recipe.
