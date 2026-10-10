# Card 32 — removestrand-scan

**Create:**
- `cmd/lyx/removestrand_test.go`
- `pattern/PATTERN-shuttle-stop.md`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `PATTERN.md`

**Intent:** D13: a scan pins `PATTERN-shuttle-stop`.
`TestShuttleStop_RemoveStrandOnlyInAllowedPackages`, marked `//lyx:guard`, walks every non-test Go file under `internal` and `cmd` through `scankit`, parses it and fails, naming the file and line, on a selector expression whose selected identifier is `RemoveStrand` in a package outside the allowlist written once in the file: `internal/reedengine`, `internal/reedcli`, `internal/shuttleengine`, `internal/loomcli`, `internal/orchcli` and `internal/testkit/shuttlefake`, each with its reason; the selector match catches a method value handed to a wrapper as well as a call, and the allowlist reports stale entries.
A fixture-tree row in the same test plants a call and a method value outside the allowlist and asserts both are found.
Every call site today is inside the allowlist, so the scan passes on landing.
`PATTERN.md`'s `PATTERN-shuttle-stop` entry names every allowed package, `reedengine`, `reedcli` and `shuttlefake` beside the existing three, gains "(test)" and links a new background file, `pattern/PATTERN-shuttle-stop.md`.
That background file takes the reasons out of the overview line: `reedengine` as reed's owner, `reedcli` as reed's own remove verb, `shuttlefake` as the test fake, why `loomcli`'s driver strand and `orchcli`'s orch strand are exempt (no attach probe or Wait reads their records), and D13's bound that the scan sees the selector, not whose strand it removes.
Per the record's amendment "PATTERN overview stays under its cap", the card keeps `PATTERN.md` at most 16200 bytes (`wc -c`) and drops no invariant: the line's added names and link are offset by the reasons moving to the background file; if the file is still above 16200 bytes, the card shortens wording in another overview line whose background file already carries the detail, never one an agent needs inline.
`cmd/lyx`'s budget row rises by one.

**ImpactSummary:** One new scan and one budget row; every production call site already sits inside the allowlist, so nothing else moves.
