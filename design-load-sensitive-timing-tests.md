# Tests that compare a measured time to a limit fail on a loaded machine

`load-sensitive-timing-tests` · Task · bug, gate, testkit · running · Drift-Baseline

Depends on: [`gate-redundancy-report`](design-gate-redundancy-report.md)

## Seen (orch, 2026-11-09)

`TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` in `internal/loomcli` failed a tier 2 gate run of `testmap-verb`: `startLLMDriverArm took 6.18s; want well under the stub's own 5s settle delay`.
The test compares the measured time of a launch with the stub's 5 s delay, so a loaded machine can push the launch past it.
That run held one of three gate slots beside two other gates, the operator's own C# builds and Cortex XDR scanning, and the failure cost a whole second gate pass.

## Idea

Tests that assert on elapsed time against a threshold fail when the machine is busy, not when the code is wrong.
List them and make each load-proof: a test that proves "does not wait on X" should block X on a release the test controls, as the same test's own deadline branch already does, instead of comparing against a fixed duration.
A loose search over `_test.go` files for elapsed-time comparisons names about twenty files, among them in `shuttleengine`, `creelengine`, `shedverbs`, `verifyrun`, `tenderengine`, `seatengine` and `reedengine`; most may only set a deadline, so the work starts by reading which ones compare a measured time to a limit.
