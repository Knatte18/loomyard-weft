```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I checked the record's claims and bounds against the code.

**Findings**

1. MEDIUM — Open risks, second bullet (the hub fan that names a deleted lens). The record says such a hub "gets `ResolveFan`'s undefined-lens refusal once the deploy lands". The real failure is wider and comes earlier.
   - `loomengine.LoadConfig` calls `lenses.ValidateFanKeys` over every perch's fan key (`internal/loomengine/config.go:1225`).
   - The template's `code_reviewer_fan` is `["review-code", ""]`. So on the Models hub, every loom config load fails while its hub `review-code` fan names `code-pattern-fit`, not only the Code-Review round. Every loom run step and every verb that loads loom config is hit, in every profile.
   - Counterexample: deploy to the OSL-1046 hub with the fan from `docs/dotnet-hub.md`. A Discussion-Write start loads loom config, `ValidateFanKeys` resolves `review-code`, and `ResolveFan` refuses the undefined lens, so the run stops before any review.
   - The open risk should state that blast radius. It should also say the hub edit must land before the deploy, not "at" it.

2. MEDIUM — D5's bound that a value "can only name a fan the guarded `lenses.yaml` defines". That bound is enforced only when a pair loads its config, not when the value is written.
   - `configreg`'s `checkLocalDocument` runs `configengine.ValidateLocal` plus loom's `ValidateLocalValues`. The second checks only `sheddriver.CheckLocalOverride` and `reviewbudget.CheckLocal`, with no fan resolution (`config.go:1158-1167`).
   - `checkProfiles` and `LayerRunSettings` use the same path (`runsettings.go:105-160`).
   - Counterexample: `lym config profiles --set light='{"loom":{"loom":{"code_reviewer_fan":"revew-code"}}}'` is admitted, as is a task `run_settings` naming a fan that doesn't exist. Every pair seeded from it then fails `LoadConfig`, so the error surfaces at seed or first step instead of at the write.
   - D5 should either add a fan check to loom's local value check (a `ValidateFanKeys` over the local document's `code_reviewer_fan`) or state that the bound holds only at load.

3. LOW — D5's bound text says nothing about a run's own agent. A task's `run_settings` write is allowed to a run's agent (the board lock exempts it), so an agent can now set its own task's code review to solo.
   - The reach is bounded by when `run_settings` are re-seeded into a live pair, which the record doesn't state.
   - The bound should say whether a mid-run `run_settings` write reaches the running pair. If it does, an agent can thin its own review.

**Claims attacked**

- D5, "configreg checks a profile block as it checks `run_settings`, through `ValidateLocal`, which refuses a guard key": held (`checkLocalDocument`, `checkProfiles`).
- D5, "`code_max_bounces` can already turn the perch off from the same layers": held (the budget keys are checked by `reviewbudget.CheckLocal` as local values, not refused as guards).
- D5, "the other five fan keys stay guards": held. The retired keys `webster_reviewer_fan` and `darn_reviewer_fan` keep their own categories, and the key move targets the reclassified key, so nothing breaks.
- D5, a value names only a lens-defined fan or `""`: broken at write time, holds at load (finding 2).
- D6, "a run already seeded keeps the block it was seeded with": held (`runprofile` doc).
- D6, a `light` pair loads `code_reviewer_fan` as one empty entry: held. `FanList` decodes a scalar `""` as `[""]`, and the key is an open map stacked whole from the local layer.
- D1/D2/D3 figures (9 of 46, 14 of 38, 7 of 38, 4 of 5): held against the audit report.
- Constraint "no file under `contracts/stencils` names a changed lens": held. The repo-wide search outside tests, `docs/research`, the template, the csharp layer and `dotnet-hub.md` found none, and Bolt's rendered stencils and specs name none either.
- Open risk, Models hub refusal: broken in scope (finding 1).
- D1, "keeps every reviewer lens inside the writer fan": held, since `board-scope` is in both fans.
- Constraint that `repo-invariants` stays right whether or not `validator-checks` lands first: held, since it names no check.

**Tool calls**

1. Read `ValidateLocalValues` and the budget key sites in `internal/loomengine/config.go`, to test D5's bound against the local value check. My context had only the `ConfigSpec` categories.
2. Searched the repo and Bolt's stencils and specs for the deleted lens names, to test the no-other-referrer constraint. My context's grep had been swamped by `way-forward` matches.
3. Searched `hubreconcile` and `ValidateFanKeys` callers, to test where an undefined lens in a hub fan surfaces. My context didn't show loom's load-time fan validation.
4. Listed the functions in `internal/configreg/runsettings.go`, to check whether profile and run-settings checks run a module load. My context showed only `checkLocalDocument`.
