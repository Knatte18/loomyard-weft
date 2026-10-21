# Fixer report — merge round 1

## Changes

- F1 (fixed as proposed): `gates` is a per-name map.
  `internal/gateslot/config.go` gains `ConfigPerNameMaps()` returning `gates`, sets `PerNameMaps` on `ConfigSpec`, and drops `gates` from `ConfigOpenMaps()`.
  `internal/configreg/configreg.go` sets `PerNameMaps` on the `gate` module; `internal/gateslot/doc.go` describes `gates` as a per-name map.
  The `gateslot` row for gate lists naming an undeclared tier now expects the template's other gates filled in.
  A new `configcli` row sets `gates.landing=[0,1]` on a hub and loads with `gateslot.LoadConfig`, asserting `landing` is `[0,1]` and `card` keeps the template list.
- F2 (fixed as proposed): the darn fixture in `preMigrationBase` plants `verify_attempts: 5`.
  `assertBoardStrippedOnce` and its doc comment expect that tuned unit.
  A new walk row, `unmigratedBoard` with `retiredKeys`, seeds the board's darn, gate and landing base files with `verify`, `go_parallel` and `publish_verify` beside template-equal units; the existing assertions require the retired keys and equal units gone, the tuned unit kept and the marker written.
- Self-check follow-up: the darn assertion in `assertBoardStrippedOnce` pins the file to the `verify_attempts: 5` line alone instead of counting colons.

- Gate repair: the gate refused the branch because the `PATTERN.md` overview grew 99 bytes past its headroom line.
  The `PATTERN-sparse-hub-base` entry now drops its parenthetical and the `lenses` exception, both already stated in `pattern/PATTERN-sparse-hub-base.md`; the overview is 15703 bytes and `lyx loom lint-pattern` is clean.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal: LOW, the darn assertion's colon count depends on comment text; fixed as proposed.

## Disputed

None.

## Deferred

None.
