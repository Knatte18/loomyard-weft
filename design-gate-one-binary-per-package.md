# One test binary per package, the tier chosen at run time

`gate-one-binary-per-package` · Task · enhancement, gate, testkit, pattern · draft

## Seen (research, landed 2026-11-01)

`docs/research/gate-fewer-binaries-trial-2026-11-01/report.md` measured three ways to cut the new test binaries a gate run builds; on OSL-1046 the virus scanner holds each new binary about 10 s, several in parallel, so a worst-case gate run there takes 12 to 13 minutes against 2 to 3 on Neve (operator, 2026-11-01).
Its verdict: one test binary per package for all tiers, the tier chosen at run time, is the cut worth making permanent.
On the module's package graph it takes a cold run from 532 test binaries to 182, and a change of mean fan-in from about 101 new binaries to about 35; on a ten-package sample every test that passed before still passed.

## Design (operator, 2026-11-01)

- The test tier is chosen at run time by `LYM_TEST_TIER`, not by build tags, so each package builds one test binary that holds every tier's tests.
- `LYM_TEST_TIER` names one tier (`1`) or a range up to a tier (`0-2`).
- Every Test function calls the tier's `Require`, which skips it when its tier is not among those named; the report's trial 2 shows the form.
- `gate.yaml` gives each tier an `env` entry setting `LYM_TEST_TIER`; the tag guards, `go vet` per tag and the gate's `tier` command follow.
  A `--upto` run may pass a range and run several tiers in one `go test` call per package where their gate slots and timeouts allow; tier2's own slots and timeouts stay as they are.
- `PATTERN-test-speed` changes from disjoint tags to the run-time selector.
- Known costs the report names: tier3 dependencies (`llmkit`) join every binary and raise its fan-in, a binary is larger, and a test that reads `LYM_TEST_TIER` loses its test-cache entry per tier.

## Shape of the run

- The tag removal and the `Require` calls across about 180 packages are made by one script over the whole module, as trial 2's `untag.sh` did, never by hand.
- When the gate then fails in many packages, the darn writer groups the failing packages into batches of a fitting size and hands each batch to a Sonnet subagent of its own, then gates the whole module once.
- Like a rename it runs with no other code-changing run in flight; a docs-only research run beside it is fine.
