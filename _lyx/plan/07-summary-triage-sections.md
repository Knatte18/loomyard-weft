# Card 7 — summary-triage-sections

**Edit:**
- `summary.go`
- `integration.go`
- `runlevel.go`
- `summary_test.go`

**Create:**
- `internal/websterengine#AppendIntegrationTriage`

**Uses:**
- `internal/websterengine#IntegrationFailure`

**Intent:** Give `summary.md`, the channel that reaches the operator and the PR, the failing identities and their tails (D11).

`AppendIntegrationFailure` gains a `regressions []IntegrationFailure` parameter after the existing ones.
Its `## Integration suite failed` section keeps today's localized-card sentence and, when `regressions` is non-empty, lists each regressing identity with its tail in a fenced block.
With nil `regressions` the section is byte-identical to today's.
Update its callers in the same card, passing nil: `BisectAndEscalate` in `integration.go` and `runIntegrationStage` in `runlevel.go`.

`AppendIntegrationTriage` appends a `## Integration suite triage` section stating the plan-level verify failed and webster's triage did not attribute the failure to this run, then one sub-list per non-empty category (flaky, pre-existing) naming each identity.
It takes identity ids only, the same lists `IntegrationTriage` carries, and writes no tails: a non-regression failure is informational, and the tails stay in the integration report.
It is a no-op when both lists are empty.
Both functions share the existing append-open-write shape; factor the open/append into one unexported helper rather than duplicating it.

The untagged `summary_test.go` gains cases asserting the exact appended text for:
a regression section with two identities and their tails;
nil regressions producing today's section unchanged;
a triage section with both categories;
a triage section with only flaky identities, carrying no pre-existing sub-list;
and both lists empty leaving the file untouched.

**ImpactSummary:** `AppendIntegrationFailure` has two callers, both passed nil here, so `summary.md` output is unchanged until card 9 supplies regressions.

**Commit:** 7: summary-triage-sections
