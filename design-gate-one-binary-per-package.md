# One test binary per package, the tier chosen at run time

`gate-one-binary-per-package` · Task · enhancement, gate, testkit, pattern · draft

Depends on: [`board-root-p0`](design-board-root-p0.md)

## Seen (research, landed 2026-11-01)

`docs/research/gate-fewer-binaries-trial-2026-11-01/report.md` measured three ways to cut the new test binaries a gate run builds, each of which costs about 10 s of scanner hold on OSL-1046.
Its verdict: one test binary per package for all tiers, the tier chosen at run time, is the cut worth making permanent.
On the module's package graph it takes a cold run from 532 test binaries to 182, and a change of mean fan-in from about 101 new binaries to about 35; on a ten-package sample every test that passed before still passed.
That is about 660 s saved per mean change on OSL-1046, and about 3600 s per change to a universally imported package.
Cutting the five universal imports out of `TestMain` saves only when one of the five changes; merging small leaf packages was not needed.

## Design (from the report, operator 2026-11-01)

- The test tier is chosen at run time by `LYM_TEST_TIER`, not by build tags, so each package builds one test binary that holds every tier's tests.
- Every Test function calls the tier's `Require`, which skips it outside its tier; the report's trial 2 shows the form.
- `gate.yaml` gives each tier an `env` entry setting `LYM_TEST_TIER`; the tag guards, `go vet` per tag and the gate's `tier` command follow.
- `PATTERN-test-speed` changes from disjoint tags to the run-time selector.
- Known costs the report names: tier3 dependencies (`llmkit`) join every binary and raise its fan-in, a binary is larger, and a test that reads `LYM_TEST_TIER` loses its test-cache entry per tier, so each tier call reruns the binary.

## Shape of the run

The `Require` call reaches the Test functions of about 180 packages: that part is mechanical and touches nearly every test file, so like a rename it runs alone, with no other run in flight.
