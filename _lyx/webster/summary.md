# driver-arbiter: arbiter verb, re-launch consumption and batten hold

Three batches ran in order and each recorded `done`.

## Batch 01: shedrun-arbiter-paths (cards 1–6), head `ba93e3252`

- `internal/shedrun` declares the arbiter's durable attempt directory and ephemeral in-flight marker paths.
- `internal/shedengine` gained the admit-checked, pause-keeping `Relaunch` and `CheckGotoTarget`.
- `internal/shedverbs` gained the `--arbiter-attempt` flag and `Hooks.ConsumeAttempt`.
- The loom config gained the `arbiter`, `arbiter_timeout_min` and `arbiter_max_spawns` keys and `ResolveArbiter`.
- `internal/loomengine/arbiter.go` holds the role, stop key, attempt, verdict, applied mark and marker records.
- `internal/shedadapters` gained `PendingEscalation`.
- Reported deviations: `internal/loomengine/driver.go`, `internal/loomengine/driver_test.go`, `internal/shedverbs/loop.go` and `internal/shedengine/goto_test.go`.
- The `internal/loomengine` test budget rose from 43 to 50.

## Batch 07: arbiter-stencil (cards 7–12), head `608debbe5`

- The arbiter's opening stencil was written and registered.
- `internal/loomcli` gained the arbiter prompt and shuttle spec, the closed eligibility list, the arbiter verb (spawn, re-run, budget, wait, timeout, verdict validation), its command wiring and the re-launch consumption hook.
- The fork also fixed a malformed `internal/loomengine` row in `cmd/lyx/testdata/test-budget.yaml` left by batch 01, and raised the `internal/loomcli` row.
- The fork read a non-empty transient class as making any error kind eligible, which the card words ambiguously.
- No deviations were reported.

## Batch 13: decision-by-arbiter (cards 13–17), head `885578a3c`

- `lyx loom decision add --by arbiter` is admitted, with its refusal-spec row.
- The driver stencil gained an `## Arbiter` section behind a new `arbiter_command` marker.
- Batten's state-changed notice is held while an arbiter is live, and the revive no longer relaunches an arbiter strand.
- The orch stencil tells the orch that an escalation now carries the arbiter's verdict.
- Reported deviation: `internal/loomcli/wiring_seams_test.go`, which needed `InnerRun.ArbiterInFlight` listed as a batten-only seam.
- The test budget rows rose to 104 and 64 for the two batten packages.

## Verify gate fixes

The first verify run failed three `internal/testkit/envkit` tests: `TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths` and `TestNilSeams_SkipsNilLegalPaths`.
Each reported `InnerRun.ArbiterInFlight` as an unfilled seam.
The fixer's reply says `FullEnv` in `internal/testkit/envkit/envkit.go` now sets that seam to a stub reporting no arbiter, with neighbouring field alignment regrouped for gofmt, and that it changed no tests.
Commit `127ff9dbf`: `fix: fill InnerRun.ArbiterInFlight in the envkit full environment`.
The fixer reports `lyx gate test --tags integration ./internal/testkit/envkit` passing.

## Process notes

Forks reported small edit-rule slips: a `cat >>` append in batch 01, a `sed -i` budget bump in batch 07 and a `sed` against `/dev/null` in batch 13. The content of each change is correct.
