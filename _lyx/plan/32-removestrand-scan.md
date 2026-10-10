# Card 32 — removestrand-scan

**Create:**
- `cmd/lyx/removestrand_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `PATTERN.md`

**Intent:** D13: a scan pins `PATTERN-shuttle-stop`.
`TestShuttleStop_RemoveStrandOnlyInAllowedPackages`, marked `//lyx:guard`, walks every non-test Go file under `internal` and `cmd` through `scankit`, parses it and fails, naming the file and line, on a selector expression whose selected identifier is `RemoveStrand` in a package outside the allowlist written once in the file: `internal/reedengine`, `internal/reedcli`, `internal/shuttleengine`, `internal/loomcli`, `internal/orchcli` and `internal/testkit/shuttlefake`, each with its reason; the selector match catches a method value handed to a wrapper as well as a call, and the allowlist reports stale entries.
A fixture-tree row in the same test plants a call and a method value outside the allowlist and asserts both are found.
Every call site today is inside the allowlist, so the scan passes on landing.
`PATTERN.md`'s `PATTERN-shuttle-stop` entry names every allowed package, `reedengine` as reed's owner, `reedcli` as reed's own remove verb and `shuttlefake` as the test fake beside the existing three, and gains "(test)".
`cmd/lyx`'s budget row rises by one.

**ImpactSummary:** One new scan and one budget row; every production call site already sits inside the allowlist, so nothing else moves.
