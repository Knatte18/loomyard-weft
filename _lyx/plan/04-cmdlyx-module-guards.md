# Card 4 — cmdlyx-module-guards

**Edit:**
- `cmd/lyx/configstrictness_test.go`
- `cmd/lyx/spawnobservability_test.go`
- `cmd/lyx/prerunlogging_test.go`
- `cmd/lyx/constraintchokepoint_test.go`
- `cmd/lyx/cwdmutation_test.go`
- `cmd/lyx/registration_test.go`
- `cmd/lyx/sandbox_coverage_test.go`
- `cmd/lyx/crosscompile_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Convert the remaining `cmd/lyx` invariant scans and root lookups to `scankit` per D2 and D4, following the overview's `scan-conversion-protocol`.
The scans (`configstrictness`, `spawnobservability`, `prerunlogging`, `constraintchokepoint`, `cwdmutation`, `registration`) take root, walk, allowlist and floor from `scankit`, drop every `go env GOMOD` exec and `exec.LookPath("go")` root-lookup skip, and gain stale-entry detection and a floor where they had none.
`registration_test.go` keeps exactly what it asserts (D5 makes it the mount oracle).
`sandbox_coverage_test.go` takes only its root from `scankit`;
its excluded-module allowlist becomes one entry per line (D8) with stale detection.
`crosscompile_test.go` (integration-tagged) takes its root from `scankit` and keeps its `exec.LookPath("go")` skip, which guards the `go build` it runs rather than a root lookup.
Stale entries removed and planted-violation proofs go in the commit body.

**ImpactSummary:** Eight `cmd/lyx` test files change scaffolding; registration and sandbox-coverage verdicts are unchanged.
