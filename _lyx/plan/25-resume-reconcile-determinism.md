# Card 25 — resume-reconcile-determinism

**Edit:**
- `internal/loomcli/smoke_prebootstrap_integration_test.go`
- `internal/loomcli/smoke_helpers_test.go`

**Intent:** Make `TestLoomResumeReconcilesTheHubConfigBeforeArming` deterministic by reproduction (D11).
Reproduce it first: run the package's integration tier at least twenty times, narrowed to the test with the per-card verify below, and name the dependency the failures show.
The record names the two to rule out first: the subprocess timeouts of a cold `lyx` under a loaded hub, and `sharedLyxBinary`'s temp dir sitting outside the test's own.
Force the named dependency to show the test failing before the fix, remove it (an injectable timeout, a build dir under the test's own tree, or whatever the reproduction names), and show the test passing after under the same repeated runs; the fix is not predicted here.
When no run reproduces the failure, the card's commit documents the attempt and the dependencies ruled out in the test file's comment, and changes nothing else.
The test stays in the integration tier.

**ImpactSummary:** Two test files in loomcli; no production symbol changes unless the reproduction names an injectable timeout, which then joins this card's targets.

**Verify:** lyx gate test --tags integration ./internal/loomcli -- -run '^TestLoomResumeReconcilesTheHubConfigBeforeArming$' -count 20
