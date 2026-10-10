# Webster round 2 — fixer report

## Fixed

### F1 — the card-gate tests run in parallel

Commit `3e0c3bf33`.
`TestCardGateCommand` and `TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach` in `internal/websterengine/cardgate_test.go` now call `t.Parallel()`, and so do every `TestCardGateCommand` subtest and every table row under `nested module`.
They touch only their own `t.TempDir()` roots, and `cardGateCommand` and `renderCardGates` read no env, cwd or logger state.
`go test ./internal/websterengine` passes.

### F2 — an absent gate.yaml has one way forward, its own spec row, and a CLI test

Commit `98556f020`.
Plan card 2 says `LoadConfig` names `lyx fabric reconcile` when the file is absent, so the loader's message stays and the CLI and the spec now agree with it:

- `internal/gateslot/config.go`: the absent-file error is now the exported sentinel `ErrConfigAbsent`, with the message unchanged.
- `internal/gatecli/run.go`: `errors.Is(err, gateslot.ErrConfigAbsent)` gives the envelope the single way forward `run "lyx fabric reconcile", then re-run the same command`.
  Any other load error keeps the `lyx config gate` way forward.
- `contracts/specs/refusal-spec.md`: the `## gate` table has a new `gate.yaml absent` row with that way forward.
  The `unreadable or invalid` row now lists only the "cannot be read", "does not parse" and "value below 1" triggers.
- `internal/gatecli/doc.go`: the Slot section states the absent-file way forward.
- `internal/gatecli/test_integration_test.go`: `TestGateTest_Scenario`'s refusal table has a new row.
  Its setup removes `gate.yaml`, its restore rewrites it, and it asserts the `lyx fabric reconcile` way-forward clause.
  This is a row in the existing test, not a new test, because the finding was a coverage gap.

Verification: `go vet` passes for `gateslot`, `gatecli` (untagged and `integration`) and `hubgeom`.
`go test` passes for `gateslot`, `gatecli`, `hubgeom`, `contracts/stencils` and `cmd/lyx`.
`go test -tags integration -run TestGateTest_Scenario ./internal/gatecli` passes, the new row included.
`lyx loom lint-comments` passes.
The installed `lyx` has no `gate` verb yet, so each run was a package-scoped `go test`, never a module-wide run.

## Disputed

None.

## Deferred

None.
