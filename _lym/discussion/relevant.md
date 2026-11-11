# Relevant files

- `internal/gatecli/scope.go` `computeScope` — forms the computed diff scope the new scope narrows
- `internal/impactset/targets.go` `DeriveTargets` — changed and importing packages from the diff
- `internal/gateslot/config.go` — validates a gate key's scope value and scoper
- `internal/gateslot/template.yaml` — the gate keys' default tiers and scopes
- `internal/gateslot/tiers.go` `FailedTestsCall` — existing anchored -run form
- `internal/gatecli/green.go` `greenRecord.newestWholeAncestor` — green entries a selected-test pass must not mark whole
- `internal/gatecli/testmap.go` — mapped tiers and staleness of maps
- `internal/gatecli/redundancy.go` — the redundancy step that reads maps and their file drift
- `internal/testcover/coveragemap.go` `ReadMap` — map record and loader
- `internal/testcover/blockkey.go` `KeyBlocks`, `CoveredKeys` — content block keys and their enclosing function
- `internal/testcover/runner.go` `Toolchain.BuildBinary` — coverage binary over a coverpkg list
- `internal/testcover/profile.go` `PackageBlocks` — filter to one package's blocks
- `internal/testcover/spawnscan.go` — the spawn verdict testmap now records
