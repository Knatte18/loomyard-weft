# Plan round 3 — fixer report

## Changes

- **F1** (MEDIUM, fixed as proposed): `_lyx/plan/09-batten-overview.md`.
  The `frame` paragraph now states that a run whose `shedrun.ReadSeed` fails is kept as a batten run with no `started_at`, sorts last by slug, and has its seed read again at the next poll since it is not cached.
  The `TestOverviewSource_ReadsThePrimesRunsFromDisk` bullet gains a row with a `seed.json` holding an unknown key, a batten status file and a pair directory, whose line follows the lines of the runs with readable seeds.
  Premise confirmed: `shedrun.List` keeps any directory whose `seed.json` reads as a file (`internal/shedrun/seed.go`), and `ReadSeed` decodes strictly.
- **F2** (LOW, fixed as proposed): `_lyx/plan/01-render-size-vocabulary.md`.
  `ValidateDisplay` now returns an error naming the display's anchor and kind, with no "same wording" claim for own-window;
  `planCells` wraps it as `render: strand <guid>: <error>`, so `Rules` and `FixedHeightPins` still name the offending strand.
  Premise confirmed: today's refusal embeds `s.GUID` (`internal/reedengine/render/rules.go`), which `ValidateDisplay(d Display)` never sees.

## Disputed

None.

## Deferred

None.
