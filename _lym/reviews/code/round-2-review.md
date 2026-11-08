---
verdict: APPROVED
---

No findings.

## Rejected

- Whether a test reads the plan directory handed to the Drift expect agent: `TestWire_DriftExpectPlanReading` in `internal/loomcli/wiring_darn_test.go` wires the sampler, darn and tapestry recipes and reads `plan_reading` from `c.env.Drift.ExpectTable`.
  It asserts the plan directory is absent for sampler and darn, present for tapestry, and that the absent text opens `none:`.
  Reverting `hasPlan` at `internal/loomcli/wiring.go:1041` to `!darnRun` would give the sampler run a plan directory, so the sampler row fails.
  Not a defect; the coverage holds.
- Whether `docs/overview.md` matches the sampler rows: its regression-rows section states that a sampler run's rows also need `drift.scope` set to `full` beside `drift.enabled`, and that `Drift-Expect` comes before `Sampler-Session` in sampler.
  Both match the entry gates and row order in `contracts/recipes/sampler-recipe.yaml`.
  Not a defect.
- Scope against the decision record: the sampler recipe gains darn's Drift rows gated by `drift.enabled` and `drift.scope == full`, so the default `off`/empty scope runs the sampler as before.
  The switch is the existing `drift.scope`, no new key.
  The Stuck route to `Sampler-Session` in `internal/driftengine/verify.go` is covered by the `AnUnexpectedCellUnderSamplersRows` row.
  The tapestry and darn gates are unchanged, and `TestDriftRows_EntryGate` pins all three recipes across scope values.
  The `lym gate test ./internal/loomcli` run passed at tier 0.
