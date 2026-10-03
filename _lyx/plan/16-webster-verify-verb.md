# Card 16 — `lyx webster verify` and the Verified-Tree Invariant

**Create:**
- `internal/webstercli/verify.go`
- `internal/webstercli/verify_integration_test.go`
- `cmd/lyx/verifiedtree_test.go`

**Edit:**
- `internal/webstercli/cli.go`
- `CONSTRAINTS.md`

**Uses:**
- `internal/verifytree/verify.go`
- `internal/websterengine/verifygate.go`
- `internal/shedrecipe/entries_gate.go`
- `internal/landingshed/verifygate.go`

**Intent:**
D15: a new verb, `lyx webster verify`, is the webster gates' self-check.
It parses the plan at `c.geom.PlanDir` (hub or standalone geometry), and calls `verifytree.Verify` with `verifytree.NewPaths(c.geom.WorktreeRoot, c.geom.VerifyDir)`, site label `webster verify` and the plan's `## verify:` command.
A pass or skip exits 0 and a pass records the tree;
a dirty tree or a failing command exits 1 with findings under the failure envelope's `findings` key (the dirty paths, or the exit code, log path and failing identities);
a plan with no `## verify:` section exits 0 with a warning, as the gate passes it.
`cli.go` registers the verb and lists it in the help text.
The verb records only after a pass of its own, because only `verifytree.Verify` writes the record.

D16: `CONSTRAINTS.md` gains the Verified-Tree Invariant:
every plan-verify site goes through `verifytree.Verify`;
none verifies a dirty tree;
a skip requires the record to name HEAD's tree and the same command;
the record is written only after a pass.
The Gate Self-Check Parity Invariant's pair list gains the webster `verify` gate and Webster-Burler's `verify` gate ↔ `lyx webster verify`: `verifytree.Verify`.
The Told-Geometry Invariant's bound-package list gains `verifytree`.

`cmd/lyx/verifiedtree_test.go` is the parity enforcement test and the invariant's tripwire, a tier-1 source scan through `scankit` like `constraintchokepoint_test.go`:
the webster gate (`internal/websterengine/verifygate.go`), the Webster-Burler gate (`internal/shedrecipe/entries_gate.go`), the verb (`internal/webstercli/verify.go`) and the landing gate (`internal/landingshed/verifygate.go`) each call `verifytree.Verify`;
no production file outside `internal/verifytree` calls `verifyrun.Run`, except `internal/websterengine/cardverify.go`, which reruns per-card `**Verify:**` commands and is no plan-verify site;
no production file names `verify-pending`.

Tests in `verify_integration_test.go` (integration tag, hub fixture as `cli_integration_test.go` builds it):
a passing plan verify exits 0 and records;
a second call on the same tree exits 0 without running;
a failing plan verify exits 1 with findings;
a dirty tree exits 1 naming the paths.

**ImpactSummary:** webster gains one verb; CONSTRAINTS.md gains one invariant and two list entries; cmd/lyx gains one source-scan guard.
