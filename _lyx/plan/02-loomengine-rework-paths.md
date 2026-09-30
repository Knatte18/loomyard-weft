# Card 2 — loomengine-rework-paths

**Edit:**
- `internal/loomengine/config.go`
- `internal/loomengine/config_test.go`
- `cmd/lyx/notransients_test.go`
- `cmd/lyx/constructoranchoring_test.go`

**Intent:**
Add the path accessors the rejection record and the rework row need, so no other package constructs them (Told-Geometry, Durable-vs-Ephemeral State and Cwd Resolution Invariants).
Ephemeral, built on `LoomScratchDir` exactly like `LoomApprovalPath`:
`LoomRejectionPath(l)` (`rejection.json`, the pending record `lyx loom reject` writes) and `LoomReworkCoveragePath(l)` (`rework-coverage.md`, the rework agent's completion file, which maps each finding to the new cards covering it).
Durable, built on `LoomDurableDir`/`LoomDurableDirRel` exactly like `LoomFrictionDir`:
`LoomReworkDir(l)` and `LoomReworkDirRel()` (`rework`), the root holding one `round-<N>` directory per rejection round.
Each new filename or segment is an unexported constant beside `loomApprovalFileName`.
Extend `config_test.go` with each accessor's expected path, add the two ephemeral accessors to `transientSet` and the durable one to the durable set in `notransients_test.go`, and pin all four in both fixtures of `constructoranchoring_test.go`, as `LoomApprovalPath` is pinned today.

**ImpactSummary:** Adds four accessors and their test rows; no existing path changes.
