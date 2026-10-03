# Card 18 — `lyx webster verify` and the Verified-Tree Invariant

**Create:**
- `internal/webstercli/verify.go`
- `internal/webstercli/verify_integration_test.go`
- `internal/webstercli/verify_parity_integration_test.go`
- `cmd/lyx/verifiedtree_test.go`

**Edit:**
- `internal/webstercli/cli.go`
- `CONSTRAINTS.md`

**Uses:**
- `internal/verifytree/verify.go`
- `internal/websterengine/verifygate.go`
- `internal/loomshed/verifygate.go`
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
The Gate Self-Check Parity Invariant's pair list gains the webster `verify` gate (`websterengine.NewVerifyGate`) and Webster-Burler's `verify` gate (`loomshed.NewVerifyGate`) ↔ `lyx webster verify`: `verifytree.Verify`.
The Told-Geometry Invariant's bound-package list gains `verifytree`.

`verify_parity_integration_test.go` is the parity check, in the shape of `loomcli`'s `TestGateParity_*`:
over one scratch hub fixture per case, it drives the verb, `loomshed.NewVerifyGate` and `websterengine.NewVerifyGate` (with a `done` `outcome.yaml` and no recorded pre-fix head), maps each onto a three-way verdict (done, stuck, error) and asserts all three agree.
Cases: a passing command, a failing command, a dirty tree, and a plan with no `## verify:` section.
It is integration-tagged because every side spawns git and a shell.

`cmd/lyx/verifiedtree_test.go` is the invariant's tripwire, a tier-1 source scan through `scankit` like `constraintchokepoint_test.go`:
the webster gate (`internal/websterengine/verifygate.go`), the Webster-Burler gate (`internal/loomshed/verifygate.go`), the verb (`internal/webstercli/verify.go`) and the landing gate (`internal/landingshed/verifygate.go`) each call `verifytree.Verify`;
no production file outside `internal/verifytree` calls `verifyrun.Run`, except `internal/websterengine/cardverify.go`, which reruns per-card `**Verify:**` commands and is no plan-verify site;
no production file names `verify-pending`.

Tests in `verify_integration_test.go` (integration tag, hub fixture as `cli_integration_test.go` builds it):
a passing plan verify exits 0 and records;
a second call on the same tree exits 0 without running;
a failing plan verify exits 1 with findings;
a dirty tree exits 1 naming the paths.

**ImpactSummary:** the verb is additive with no existing caller; the invariant and the tripwire bind every future plan-verify site, failing one that bypasses `verifytree.Verify` or brings back a marker; no existing code changes.
