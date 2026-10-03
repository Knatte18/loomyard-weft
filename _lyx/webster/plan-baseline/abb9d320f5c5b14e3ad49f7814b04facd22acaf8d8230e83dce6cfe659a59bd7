# Card 3 — cmdlyx-git-guards

**Edit:**
- `cmd/lyx/ghguard_test.go`
- `cmd/lyx/boardguard_test.go`
- `cmd/lyx/rawgitmutation_test.go`
- `cmd/lyx/destructiveguard_test.go`
- `cmd/lyx/uncontainedwrite_test.go`
- `cmd/lyx/checkedcall_test.go`
- `cmd/lyx/gitrepoboundary_test.go`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Convert the `cmd/lyx` git and fabric guards to `scankit` per D2 and D4, following the overview's `scan-conversion-protocol`.
Each drops its `exec.LookPath("go")` skip and `go env GOMOD` exec and takes root, walk, allowlist and floor from `scankit`;
its matching logic, token set, allowlist entries and floor stay verbatim.
Each gains stale-entry detection on whatever allowlist it carries.
`gitrepoboundary_test.go`'s pinned-method set is the first allowlist rewritten one entry per line with no aligned values (D8);
every other allowlist touched here follows.
Stale entries the tightening finds are removed and named in the commit body;
each scan is proved by one planted violation, also named there.
The `allowedSpawners` entries in `cmd/lyx/tierpurity_test.go` that existed only for these files' `go env` exec go stale here and are removed in card 5, which adds the stale detection that finds them.

**ImpactSummary:** Seven test-only scans in `cmd/lyx` change scaffolding; their verdicts on the real tree are unchanged.
