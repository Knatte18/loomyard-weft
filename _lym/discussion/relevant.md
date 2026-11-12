# Relevant files

- `internal/testcover/upkeep.go` `PlanUpkeep` — the plan that marks a current map changed when its commit moves; D7 changes it
- `internal/testcover/coveragemap.go` `Dir`, `ReadMap`, `WriteMap`, `EnvironmentStamp` — the map's path, format and stamp; D1 and D8
- `internal/testcover/testselect.go` `MissEnvironment` — the miss a differing stamp gives, and the marker check D7 relies on; D6 and D7
- `internal/gatecli/testmap.go` `testMapLocation` — the prime-only refusal D9 widens and D6 extends
- `internal/gatecli/testmappackage.go` `stampEnvironment` — what the tiers and hub env digests hash; D6
- `internal/preflight/preflight.go` — the worktree-clean check D5 exempts `_lym/testcover/` from
- `internal/gatecli/coveragescope.go` — the gate's selector, which reads the maps; D1
- `internal/gatecli/redundant.go` — `lym gate redundant`, a map reader; D1
- `internal/dobbyengine/template.yaml` — the `test_map_upkeep` default; D10
- `internal/dobbyengine/testmap.go` — the dobby's pass starter and its exit-record read; D4 and D10
- `internal/beamcli/dobbydeps.go` — wires the starter with the bolt maps dir; D1
- `internal/fabricengine/forkrecords.go` — the weft fork that drops only run records; D2
- `internal/fabricengine/commitweftpaths.go` `CommitRecordsPaths`, `CommitAnchoredPaths` — the scoped weft commit; D4
- `internal/fabricengine/junctionnames.go` — the structural tracked and never-tracked dirs; D1
- `internal/lymdirs/dirs.go` — names `testcover` and the exit file; D1
- `pattern/PATTERN-test-speed.md` — describes the maps and the upkeep pass; review criterion
