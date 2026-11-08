# Round 1 fixer-report

## Changes

- F1 (MEDIUM), fixed as proposed, commit `7fecfc73`.
  `internal/loomcli/wiring_darn_test.go` gains `TestWire_DriftExpectPlanReading`, a table over the sampler, darn and tapestry recipes.
  It wires each and asserts the expect table's `plan_reading` through `c.env.Drift.ExpectTable`:
  empty plan directory (the "none:" reading) for sampler and darn, `planparser.PlanDir` of the anchor for tapestry.
  The assertion goes through `ExpectTable` because `driftDeps` keeps the plan directory unexported;
  `driftengine/seats.go` derives `plan_reading` only from that value.
  The old `!darnRun` condition would give a sampler run a plan directory and fail the sampler row.
- F2 (LOW), fixed as proposed, commit `56f915db`.
  `docs/overview.md` regression-rows section says a sampler run's rows also need `drift.scope` set to `full`,
  and the `Drift-Expect` ordering sentence adds "before `Sampler-Session` in sampler".
- Self-check follow-ups, commit after `56f915db`:
  the skipped-rows sentence says "fails either gate", and the test file header lists the plan-reading test.

`lym gate test ./internal/loomcli` passes after every change.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the file header of `wiring_darn_test.go` omitted the new test; fixed as proposed.
- fix-proposal: LOW, "leaves the gate off" read as one gate after F2's edit; fixed as proposed ("fails either gate").
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
