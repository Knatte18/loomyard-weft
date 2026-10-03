# Card 34 — battenrecipe-envkit

**Edit:**
- `battenrecipe/fixture_test.go`

**Uses:**
- `testkit/envkit/envkit.go`

**Intent:**
Replace battenrecipe's full `Env` builder with `envkit.FullEnv` (#324 §3).

`testEnv` keeps its signature, returning `(shedrecipe.Env, shedbuild.ShedPaths)`, but builds the `Env` from `envkit.FullEnv(t)` and overrides only what batten's recipe tests need.
Partial `Env` literals elsewhere (`battencli/run_test.go`, the `loomCLI` literals in `loomcli`'s parity and validate tests) set two or three fields on purpose and stay literals: they are not full builders.
Apply D10 to every touched test.

**ImpactSummary:** One untagged battenrecipe fixture switches to the shared full Env; batten recipe tests are otherwise unchanged.
