# Test tiers: every hub test run goes through `lyx gate test`

Every batch reached `status: done`; the last batch head is `97f2f012`.
Batch 11 needed one recovery; no other batch did.
No digest carried `departures` or `warnings`.
The `deviations` lists below are the files each digest reported outside the batch's own targets, all informational.

## Batches

- **01 open-map-flow-values** (cards 1–3): `--set` writes a whole open-map entry as a YAML flow mapping or list.
  `gate.yaml` declares tiers, gate lists, scoper, guard tier and `refuse_raw`, with load and use-time refusals.
  The slot pool takes several slots with an ordered drain.
  Deviations: `internal/configreg/configreg.go`, `internal/gateslot/{config,pool,tiers}.go` and their tests, `internal/yamlengine/set.go` and its test.
- **04 gate-run-log** (cards 4–7): the per-tier run record and log, the timeout and summary lines, the `go` scoper's impacted targets, the card-scope package rule in `planparser`, and the `go-json` and `vstest` counters.
  Deviations: `internal/gatecli/counters.go` and its test, `internal/gateslot/runlog.go` and its test, `internal/impactset/{derive,targets}.go`, `internal/planparser/targetdirs.go`.
- **08 gate-tier-verb** (cards 8–9): `lyx gate test` runs tiers over explicit targets or `--all`, plus the computed, `--base` and `--card` scopes, per-module runs, the empty set and the guard step.
  Deviations: `internal/gatecli/{cli,run,scope,tierrun,tiers}.go` and their tests.
- **10 round-gate-tiers**: the round and darn writer gates run the `round` gate, and the failure record names its tier.
  Deviations: `internal/impactset/derive.go` and its integration test, `internal/loomcli/roundgate.go`, `internal/loomshed/{publishfailure,verifygate}.go` and tests, `internal/shedrecipe/{entries_gate.go,fixture_test.go}`, `internal/testkit/envkit/envkit.go`, `internal/verifytree/failure.go`.
- **11 card-verify-runner**: the shared card `**Verify:**` runner, `lyx gate card-verify` and `HolderEnv`.
  The first `record-batch` failed its done-check, because `TestRunner` and `TestIsGateCall` did not resolve in `internal/cardverify`.
  A backgrounded `recover-batch 11` ended `done` at `01a4b023`.
  Deviations: `internal/cardverify/{runner,runner_test}.go`, `internal/gatecli/cardverify.go`, `pattern/PATTERN-verified-tree.md`.
- **12 webster-final-gate**: webster's plan-level gate and `lyx webster verify` run the `final` gate.
  Deviations: `internal/loomcli/wiring_test.go`, `internal/webstercli/verify_integration_test.go`, `internal/websterengine/verifygate.go` and its integration test.
- **13 landing-gates**: Merge-In, Finalize and Publish run `landing` and `publish`; `publish_verify` and darn `verify` retire.
  Deviations: `internal/darnengine/config.go` and its test, `internal/landingshed/*`, `internal/loomcli/*` wiring and tests, `internal/testkit/envkit/envkit.go`.
- **14 verifytree-unslotted**: `verifytree.Verify` drops its slot and timeout.
  Deviations: `internal/landingshed/verifygate_test.go`, `internal/loomcli/landingdeps_verify_test.go`, `internal/loomshed/verifygate_integration_test.go`, `internal/shedrecipe/seam_enforcement_test.go`, `internal/verifytree/verify.go`, `internal/websterengine/verifygate*.go` and tests.
- **15 webster-prompt-gates**: webster prompts render `--card` gate calls and `lyx gate card-verify`.
  Deviations: `internal/webstercli/{beginbatch,recoverbatch}.go` and tests, `internal/websterengine/*_test.go`.
- **16 retire-verify-tiers**: prompts render the hub's tiers from `gate.yaml`; `verify_tiers` and the plan verify section retire.
  Deviations: `internal/languageconfig/{config,template}.go`, `internal/loomcli/cli.go`, `internal/shedadapters/bouncer.go`, `internal/webstercli/{cli,wiring}.go`, `internal/websterengine/render.go` and tests.
- **17 refuse-raw-guard**: the shell guard refuses the hub's `refuse_raw` commands.
  Deviations: `internal/shellguard/rows.go`, `internal/shuttleengine/claudeengine/settings.go`, its test and golden file.
- **18 plan-gate-verify-checks**: the plan gate judges `**Verify:**` lines against `refuse_raw` and `--wait`.
  Deviations: `internal/loomcli/parity_test.go`, `internal/loomshed/plancontext.go`.
- **19 non-tier-holders**: non-tier holders set their own flags, `go_parallel` retires, and PATTERN and the push rule move.
  Deviations: `cmd/testtiming/redundancy.go` and its new integration test, `internal/gateslot/{config,pool}.go` and test, `internal/planglyph/typesload.go`, `internal/standalonegeom/standalonegeom_test.go`.

## Scope guard

The glyph scope guard listed only the files above as changed outside the batches' targets.
No batch reported a deviation as a reason to stop.

## Verify gate findings

Attempt 1 of 3 failed on `internal/configsync.TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate/landing_absent_with_a_prime_copy_is_seeded_from_the_prime`.
The hub's seeded `landing.yaml` did not contain the expected `go test -tags tmux ./...` line.
The findings file records that no recorded card touched a failing package.


## Verify gate fix commits

The verify-fix fork's commits above the pre-fix head, and the PATTERN departures their messages declare.

- `bf5edc9d14c6c004ba83172d4e6b728b3594ab78` fix: seed-from-prime landing row uses a live key, not retired publish_verify
