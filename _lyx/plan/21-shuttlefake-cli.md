# Card 21 — shuttlefake-cli

**Edit:**
- `shuttlecli/cli_test.go`
- `loomcli/smoke_attachprobe_test.go`
- `loomcli/smoke_gate_test.go`

**Uses:**
- `testkit/shuttlefake/shuttlefake.go`

**Intent:**
Finish #324 §1: the remaining consumer `Engine`/`ReedOps` fakes move onto `shuttlefake`, leaving `internal/shuttleengine`'s cycle-forced in-package fake as the only other copy.

`shuttlecli/cli_test.go`: `specCapturingEngine`, `preparingEngine`, `noopReed` and `statusFailingReed` become `shuttlefake.Engine`/`shuttlefake.Reed` values with the matching func fields (`PrepareFn` capturing the spec, a `StatusErr`, an `AddStrand` override).
`loomcli/smoke_attachprobe_test.go`'s `shellLaunchEngine` and `smoke_gate_test.go`'s `gateRepromptReadEngine` embed `shuttlefake.Engine` and override `Prepare` (and only what else they script), deleting their inert method sets.

Every scripted value and assertion stays.
Apply D10 to every touched test.

**ImpactSummary:** One untagged shuttlecli test file and two loomcli smoke files drop their hand-written seam fakes; behaviour under test is unchanged.

**Verify:** go test -tags smoke ./internal/loomcli/...
