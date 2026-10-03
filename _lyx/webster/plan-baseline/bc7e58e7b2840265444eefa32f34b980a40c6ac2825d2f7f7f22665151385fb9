# Card 17 — retire the integration stage, bisect, triage baseline and integration-fix strand

**Delete:**
- `internal/websterengine/integration.go`
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
- `internal/websterengine/verifygatereport.go`
- `internal/websterengine/verifygatereport_test.go`
- `internal/websterengine/render.go`
- `internal/websterengine/template_test.go`
- `internal/websterengine/rebaseline.go`
- `internal/websterengine/suspect.go`
- `internal/websterengine/suspect_test.go`
- `internal/websterengine/gitwrap.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/doc.go`
- `cmd/lyx/rawgitmutation_test.go`
- `internal/fabriccli/merge_verbs.go`
- `internal/fabriccli/mergein_webster_integration_test.go`
- `internal/verifyrun/run.go`
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
Remove what cards 15 and 16 left without a caller, per the decision record's retiring list.
Batch 15 already deleted `internal/websterengine/integration_test.go`, so this card does not list it.

Deleted outright:
`runIntegrationStage`, `settleInterruptedFix`, `recordTriageResult`, `failuresByID` and `batchStartSHAs` from `runlevel.go`;
`accumulatedCardSHAs` stays, since the card hint (card 14) reads it;
`AwaitIntegration`, `bisect`, `LocalizeIntegrationFailure`, `BisectAndEscalate`, `RecordIntegrationFailure`, `runVerifyCapture`, `FabricBisector` and the integration report and log paths (all of `integration.go`);
all of `integrationfix.go` and its `FixStarter`;
the triage baseline (`triageIntegrationFailure`, the verify-chain instrumentation, `earliestCommit`, `runAtBaseline`) from `triage.go`;
`AppendIntegrationFailure` and `AppendIntegrationFix` from `summary.go`;
`RenderIntegrationPrompt`, `RenderIntegrationFixPrompt`, `IntegrationTemplate` and `IntegrationFixTemplate` from `render.go`;
the two integration stencils and their `stencils.go` embeds and registry rows;
`integrationFixHeadRefusal` and `commitParentCount` from `gitwrap.go`, whose last caller outside it was `integrationfix.go` (`commitsBetween` stays under card 15's `commitsSince`, and `commitChangedPaths` under the card hint).

Kept and narrowed:
the gate's flaky helpers in `triage.go` and `summary.go` lose their pre-existing branches and take the flaky identities directly;
`IntegrationFailure` moves into `verifyparse.go` as `VerifyFailure`, and its users (`parseVerifyFailures`, `verifygate.go`, `verifygatereport.go`) follow.

State and records:
`State.IntegrationFix` and `IntegrationFixState` go, and so does the reserved `-1` batch key with its skips in `rebaseline.go` and `suspect.go`.
No migration: `state.json` decoding ignores the retired `integrationFix` field, and a run in flight with an `IntegrationFix` record or an `integration.yaml` report restarts with `--fresh`.
`reclaimEntryTimeStrands` stops looking for a fix strand.

Wiring:
`RunDeps` loses `Clock`, `OpenBisector` and `FixStarter`;
`webstercli` drops its `fixStarter` view, the `runnerFixStarter` adapter and the bisector opener, and `loomcli` drops its `OpenBisector` closure;
`shedrecipe`'s Webster entry doc and `envkit`'s nil-legal list drop the retired fields.
`doc.go` drops the remaining bisect and fix-strand text, including the two known-limit sentences about bisect.

Stale bisect text elsewhere:
`fabriccli`'s `websterInFlightWarning` and the merge-in help text drop their "integration bisect over earlier card SHAs runs on pre-merge trees" clause and keep the record-batch clause, so merge-in no longer describes a retired mechanism to Merriam or the operator;
`mergein_webster_integration_test.go`'s `assertOneWebsterWarning` stops asserting `pre-merge trees`.
`cmd/lyx/rawgitmutation_test.go`'s header comment rewords its verb-name example so it names no retired interface (`FabricBisector`, and the already-gone `FabricResetter`), and the `gitwrap.go` allowlist reason names card 15's fix-commit walk beside the reconcile walk as a `CommitParents` reader.
`verifyrun/run.go`'s package doc names its consumers as plan-verify through `verifytree` and the per-card `**Verify:**` rerun;
the CardSHAs comment in `recoverbatch.go` and the `CardSHAs` field doc in `state.go` name the card hint where they name bisect.

Tests:
`state_test.go` pins that a `state.json` carrying the retired `integrationFix` field and a `-1` batch record still loads.
The remaining edited test files drop their cases for retired symbols and follow the `VerifyFailure` rename.

**ImpactSummary:** pure removal of code cards 15 and 16 left without a production caller, so no surviving behavior changes; the reach outside websterengine is wiring (webstercli, loomcli, shedrecipe, envkit) plus comment and message text (fabriccli, verifyrun, one cmd/lyx guard comment), and the user-visible effects are that a run in flight with a fix record or `integration.yaml` needs `--fresh` and that merge-in's webster warning loses its bisect clause.
