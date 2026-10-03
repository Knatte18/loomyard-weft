# Card 15 — retire the integration stage, bisect, triage baseline and integration-fix strand

**Delete:**
- `internal/websterengine/integration.go`
- `internal/websterengine/integration_test.go`
- `internal/websterengine/integrationfix.go`
- `internal/websterengine/integrationfix_test.go`
- `internal/websterengine/integrationreport.go`
- `internal/websterengine/integrationreport_test.go`
- `internal/websterengine/bisect_test.go`
- `internal/websterengine/runverify_test.go`
- `contracts/stencils/webster/webster-template-integration.md`
- `contracts/stencils/webster/webster-template-integration-fix.md`

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/state.go`
- `internal/websterengine/state_test.go`
- `internal/websterengine/triage.go`
- `internal/websterengine/triage_test.go`
- `internal/websterengine/summary.go`
- `internal/websterengine/summary_test.go`
- `internal/websterengine/verifyparse.go`
- `internal/websterengine/verifyparse_test.go`
- `internal/websterengine/verifygate.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/render.go`
- `internal/websterengine/template_test.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/suspect.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/doc.go`
- `contracts/stencils/stencils.go`
- `contracts/stencils/claims_test.go`
- `internal/webstercli/cli.go`
- `internal/webstercli/run.go`
- `internal/webstercli/wiring.go`
- `internal/webstercli/cli_test.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`
- `internal/shedrecipe/entries_simple.go`
- `internal/shedrecipe/entries_simple_test.go`
- `internal/testkit/envkit/envkit.go`

**Intent:**
Remove what card 14 left without a caller, per the decision record's retiring list.

Deleted outright:
`runIntegrationStage`, `settleInterruptedFix`, `recordTriageResult`, `failuresByID`, `accumulatedCardSHAs` and `batchStartSHAs` from `runlevel.go`;
`AwaitIntegration`, `bisect`, `LocalizeIntegrationFailure`, `BisectAndEscalate`, `RecordIntegrationFailure`, `runVerifyCapture`, `FabricBisector` and the integration report and log paths (all of `integration.go`);
all of `integrationfix.go` and its `FixStarter`;
the triage baseline (`triageIntegrationFailure`, the verify-chain instrumentation, `earliestCommit`, `runAtBaseline`) from `triage.go`;
`AppendIntegrationFailure` and `AppendIntegrationFix` from `summary.go`;
`RenderIntegrationPrompt`, `RenderIntegrationFixPrompt`, `IntegrationTemplate` and `IntegrationFixTemplate` from `render.go`;
the two integration stencils and their `stencils.go` embeds and registry rows.

Kept and narrowed:
the gate's flaky helpers in `triage.go` and `summary.go` lose their pre-existing branches and take the flaky identities directly;
`IntegrationFailure` moves into `verifyparse.go` as `VerifyFailure`, and its users (`parseVerifyFailures`, `verifygate.go`) follow.

State and records:
`State.IntegrationFix` and `IntegrationFixState` go, and so does the reserved `-1` batch key with its skips in `rebaseline.go` and `suspect.go`.
No migration: `state.json` decoding ignores the retired `integrationFix` field, and a run in flight with an `IntegrationFix` record or an `integration.yaml` report restarts with `--fresh`.
`reclaimEntryTimeStrands` stops looking for a fix strand.

Wiring:
`RunDeps` loses `Clock`, `OpenBisector` and `FixStarter`;
`webstercli` drops its `fixStarter` view, the `runnerFixStarter` adapter and the bisector opener, and `loomcli` drops its `OpenBisector` closure;
`shedrecipe`'s Webster entry doc and `envkit`'s nil-legal list drop the retired fields.
`doc.go` drops the remaining bisect and fix-strand text, including the two known-limit sentences about bisect.

Tests:
`state_test.go` pins that a `state.json` carrying the retired `integrationFix` field and a `-1` batch record still loads.
The remaining edited test files drop their cases for retired symbols and follow the `VerifyFailure` rename.

**ImpactSummary:** websterengine loses its integration stage, bisect, triage baseline and fix strand; webstercli, loomcli, shedrecipe and envkit lose the wiring for them; two stencils retire.
