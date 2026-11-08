# Relevant files

- `pattern/PATTERN-test-speed.md` — the test economy rules, the `-redundancy` report rules and the run-through-the-gate exceptions D9 updates
- `cmd/testtiming/redundancy.go` `measurePackage`, `runAlone`, `classify` — today's per-test coverage runner and verdict the engine takes over
- `cmd/testtiming/spawnscan.go` — the out-of-process scan that decides eligibility
- `cmd/testtiming/keep.go` `scanKeeps` — reads `//testtiming:keep` markers
- `cmd/testtiming/splice.go` `spliceReport` — splices `-pkg` sections into the report
- `docs/research/speed/fabric-test-redundancy-2026-11-08/report.md` — the package-scoped study, its numbers and its limits
- `docs/benchmarks/test-redundancy.md` — the module-wide report that lists no fabricengine candidate
- `internal/gateslot/template.yaml` — gate keys and their `lints` list, the model for `reports`
- `internal/gatecli/green.go` — the per-worktree gate record and its directory
- `internal/dobbyengine/tidy.go` `countLandings`, `tidyStep` — the dobby's tick step and state record D6's refresh step sits beside
