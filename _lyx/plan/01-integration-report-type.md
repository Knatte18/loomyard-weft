# Card 1 — integration-report-type

**Edit:**
- `report.go`

**Create:**
- `integrationreport.go`
- `integrationreport_test.go`
- `internal/websterengine#IntegrationReport`
- `internal/websterengine#IntegrationFailure`
- `internal/websterengine#IntegrationTriage`
- `internal/websterengine#ParseIntegrationReport`
- `internal/websterengine#WriteIntegrationReport`

**Intent:** Give the integration report its own parse type so Go can add triage fields to it while batch reports keep their strict three-field shape (D10).

`IntegrationReport` carries the fork-written `status`, `head_sha`, and `deviations` fields with the same YAML keys as `Report`, plus two Go-written optional fields:
`failures` (`[]IntegrationFailure`, `omitempty`) and `triage` (`*IntegrationTriage`, `omitempty`).

`IntegrationFailure` holds `id`, `kind`, and `tail` (a block-literal string).
Declare the three kind values as exported constants beside it: `test`, `package`, `opaque`.

`IntegrationTriage` holds `verdict`, `rerun` (`passed` | `failed`), `baseline_sha` (`omitempty`), and the three identity lists `flaky`, `pre_existing`, `regressions` (each `omitempty`).
Declare the three verdict values (`flaky`, `pre-existing`, `regression`) and the two rerun values as exported constants.

`ParseIntegrationReport` decodes strictly (`KnownFields(true)`) and applies the same `status` and `head_sha` validation `ParseReport` applies;
share that validation through one unexported helper in `report.go` rather than duplicating it, so the two parsers cannot drift.
A fork-only report (no `failures`, no `triage`) parses cleanly.
`WriteIntegrationReport` marshals the whole struct to `path`, mirroring `WriteReport`.

The untagged `integrationreport_test.go` (package `websterengine_test`) covers:
a round trip with populated failures and triage;
a fork-only report parsing with nil `Failures` and `Triage`;
an unknown key rejected;
a bad status and a missing `head_sha` each rejected;
and `ParseReport` still rejecting a batch report that carries a `failures` key, pinning that the batch shape stays strict.

**ImpactSummary:** `ParseReport`'s status/head_sha checks move into a shared helper with unchanged behavior; its callers (record-batch, recover-batch, the integration stage) see no difference.

**Commit:** 1: integration-report-type
