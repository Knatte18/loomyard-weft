# Card 2 — rework-model-key

**Create:**
- `plan:internal/loomengine#ResolveRework` -> `func ResolveRework(cfg Config, reg modelspec.Registry) (burlerengine.ModelChoice, error)`

**Edit:**
- `internal/loomengine#ReworkSpec`
- `internal/loomcli#loomCLI.wire`
- `internal/loomengine#TestReworkSpec`
- `internal/loomengine#TestResolveReviewAndJudge_MalformedSpec`

**Intent:** Implement D6.
`ResolveRework` parses and resolves `cfg.Rework` when non-empty, else `cfg.Plan`, through the registry, and returns the model, effort and version as a `burlerengine.ModelChoice`; an error names the `rework` role and the key the value came from.
`ReworkSpec` calls it instead of resolving `cfg.Plan` itself, and its file header comment stops saying rework reuses the plan role's model-spec: it reuses the plan timeout and takes `rework`, which falls back to `plan`.
`loomCLI.wire` calls `ResolveRework` beside `ResolveReview` and `ResolveJudge` and returns its error, so an unknown alias in `rework` fails the run at start, before round 1, like every other loom model key; the value is discarded there, since `ReworkSpec` resolves again per Call.
`TestReworkSpec` becomes two subtests over one assertion body: `rework` empty spawns on `plan`'s resolved model and effort; `rework` set spawns on its own model and effort while every told path and marker is still filled.
`TestResolveReviewAndJudge_MalformedSpec` gains a row for `rework` with an ungrammatical value, asserting the error names the role.

**ImpactSummary:** One new resolver in loomengine with two callers, ReworkSpec and loomcli's wire; PR-Rework's spawned model changes only when the operator sets the new key.
