# A gate run keeps the agent's prompt cache warm

All four batches (cards 1 to 15) were recorded `done`, with no renames and no `Pattern-Departure:` trailers.
The digests list deviations, which are files changed outside a batch's own targets; they are informational only.

## Batch 01, shuttle-max-reruns-key (head 56481af1)

Cards 1 to 5.
They add the `keep_warm_max_reruns` break-even in `shuttle.yaml`, the `claude.yaml` keep-warm caps with their validation, and the engine's `KeepWarmCaps` method.
`shuttleengines.KeepWarm` now resolves a strand's cap with a fallback.
`lym shuttle await` defaults to the resolved cap, and the texts say 3600000 ms.
`verifyrun` callers run their commands without the strand.

Deviations:
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`
- `internal/shuttleengine/engine.go`

## Batch 06, verifyrun-tied-line (head 565a80d4)

Cards 6 to 10.
`verifyrun.RunTied` ties a line's shell to its parent and reports its group.
`cardverify.Runner` takes a context and reports its phase and line tree.
`runproc.RoleGate` is added.
Run records carry a call id and an abandoned mark, counted apart in the report.
The hidden `run-detached` verb, its run directory and its lease are in place.

Deviations:
- `internal/cardverify/runner.go`
- `internal/cardverify/runner_test.go`
- `internal/gatecli/detachrun_test.go`
- `internal/gatecli/report.go`
- `internal/gatecli/test_integration_test.go`
- `internal/verifyrun/run.go`
- `internal/verifyrun/run_tied_integration_test.go`
- `internal/verifyrun/testmain_test.go`

## Batch 11, gate-capped-front (head 784a4182)

Cards 11 and 12.
The capped front of `lym gate test` chooses a cap, joins or starts a detached runner, and answers.
A `--gate` key past the break-even waits with the long cap.

Deviations:
- `internal/gatecli/detachfront_integration_test.go`
- `internal/gatecli/detachfront_test.go`
- `internal/gatecli/test_integration_test.go`

## Batch 13, card-verify-capped (head b85254c1)

Cards 13 to 15.
`lym gate card-verify` takes the capped form.
Gate calls rendered for agents carry no `--wait`.
The gate rule, the implementer stencil and the darn and drift stencils read `done: false` as run again.

Deviations:
- `internal/gatecli/cardverify.go`
- `internal/gatecli/cardverify_integration_test.go`
- `internal/gatecli/detachfront.go`
- `internal/gatecli/detachfront_integration_test.go`
- `internal/gatecli/detachrun.go`
- `internal/gateslot/tiers.go`

## Verify gate findings

The plan-level verify failed on attempt 1 of 2.
The failing identity was `github.com/Knatte18/loomyard/internal/shuttleengine/claudeengine.TestBuildSettings_MatchesGolden`.
The findings file noted that no recorded card touched a failing package.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `f958a72d1e9584634dd6ef613ece080dcfbbe39e` fix: update settings golden to the done: false run-again gate rule text


## Background shells at the run's end

Master's run ended while these background shells were still outstanding, and the run never read their results.

- `git add internal/shuttleengine/claudeengine/testdata/settings-golden.json && git commit -q -m "fix: update settings golden to the done: false run-again gate rule text

Co-Authored-By: Claude Sonnet 5.5 <noreply@anthropic.com>" && git log --oneline -1 && lym gate test --wait --gate webster-fix 2>&1 | tail -15` (payload signal, outstanding 39s)
