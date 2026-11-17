# Round 3 fixer-report — plan

## Changes

- F1, fixed as proposed: `_lym/plan/01-gate-test-map-mode.md`.
  The `testMapRun` drop list adds `inert`, noting `headSelector` was its one reader;
  the `testMapRun.prepare` paragraph drops the `r.inert = cfg.Inert` assignment.
  Premise checked: `internal/gatecli/testmap.go:363` is the one write and `internal/gatecli/testmapplan.go:148`, inside `headSelector`, the one read.
- F2, fixed as proposed: `_lym/plan/03-map-run-tier-bounds.md`.
  The `mapSideBySide` bullet takes the tier's `Parallel` as the config gives it, without the `runtime.NumCPU()` fallback;
  the `TestMapSideBySide` row for a `Parallel` of 0 is gone.
  Premise checked: `internal/gateslot/tiers.go:191` defaults `Parallel` to 1 and `:236` refuses a value below 1.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
