# Sampler, an interactive loom recipe

All five batches reached `done`; the scope guard found no blocking deviation, no batch reported a pattern departure or a rename, and no `record-batch` returned a warning.

## 01-sampler-config

Built the hub-wide `sampler.yaml` module in `internal/samplerengine` (config, record with done flag and entry stamp, session spec and the `sampler-template-agent` stencil) and loomshed's Sampler-Session producer.
Head `959521fd`.
Deviations the digest listed: `cmd/lym/configstrictness_test.go` (new `configengine.Load` call site added to the strict set), `internal/loomshed/samplersession.go`, `internal/samplerengine/config.go`, `record.go` and `spec.go`.
The fork noted that the producer's halt reasons carry a literal `&lt;slug&gt;` placeholder in the `lym batten cancel` hint, because `SamplerDeps` has no slug field.

## 05-sampler-registry-entry

Registered `SamplerSession` as a loom build engine with `Env.Sampler` and added the sampler recipe file and its name in the recipe vocabulary.
Head `7f1f5d0d`.
Deviations: `internal/loomrecipe/darnrecipe.go` (verify-gated rows now track a build row separately from its verify gate, so the sampler recipe keeps `Code-Burler-Fix`), `internal/shedrecipe/entries_samplersession.go`, `internal/shedrecipe/fixture_test.go` and `internal/testkit/envkit/envkit.go` (no-op `SamplerSessions` fakes).
The fork also added the `InterruptPolicies` entry for `NameSamplerSession` that batch 01 left out.

## 07-sampler-vocabulary

Carried `sampler` into batten seeding, the board, run profiles and run descriptions.
Head `fdb41bb0`.
Deviation: `internal/loomcli/arm.go` gained the `samplerSessionEngine` name constant.

## 08-rows-keyed-wiring

Keyed tree-scope, record and perch-off wiring on the recipe's rows, wired the Sampler-Session seams, added `lym loom sampler-done` and widened `decision add` to admit the sampler session with `--by operator`.
Head `e6c65862`.
Deviations: `internal/loomcli/decision.go`, `internal/loomcli/samplerdone.go` and `internal/loomcli/samplerwiring.go`.

## 12-operator-wait-rows

Added one list of operator-wait rows, `OperatorWaitRows` and `WaitsOnOperator` in `internal/loomshed`, read by dobby's `childAtWaitingRow` and by the step watchdog's `agentActivityFor`.
Head `c661d15d`.
Deviation: `internal/loomcli/looparm.go`, which gained the `runWaitsOnOperator` helper.

Each fork's batch gate passed on tiers 0 and 1; tiers 2 and 3 did not run.

## Verify gate findings

The plan-level verify failed on the `github.com/Knatte18/loomyard/cmd/lym` package (verify attempt 1 of 2); no recorded card touched a failing package.
The findings file's log tail showed the `TestGofmt_ModuleFilesAreFormatted` subtests, and the full log is `.lym/verify/verify-2.log`.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `13f6dd0823c9bcb7fb3384ba421c97eea1821c25` fix: shorten lym config Long under the byte cap the sampler module name pushed it past
