# Card 8 — integration-fix-record

**Create:**
- `internal/websterengine#IntegrationFixRecord`
- `internal/websterengine#AppendIntegrationFix`

**Edit:**
- `internal/websterengine/integrationreport.go`
- `internal/websterengine/summary.go`
- `internal/websterengine/integrationreport_test.go`
- `internal/websterengine/summary_test.go`
- `contracts/stencils/landing/landing-template-describe.md`

**Prosa:**
- `contracts/specs/webster-spec.md`

**Intent:**
Give the integration-fix attempt its record in the integration report and in summary.md (#332b, decision D5).

In `integrationreport.go`, add `IntegrationFixRecord` with YAML fields:
- `result`: `fixed`, `failed`, `timeout`, `refused` or `spent`;
- `detail`: the reason behind a result other than `fixed`;
- `pre_fix_head`: the HEAD the attempt started from;
- `commits`: the strand's commits after `pre_fix_head`, oldest first;
- `cleared`: the regressing identities the post-fix triage no longer reports;
- `remaining`: the regressing identities it still reports.

`IntegrationReport` gains `Fix *IntegrationFixRecord` (`yaml:"fix,omitempty"`), Go-written like `Failures` and `Triage`.
A fork-only report still parses; a report carrying the field round-trips through `WriteIntegrationReport` and `ParseIntegrationReport`.

In `summary.go`, add `AppendIntegrationFix`, which appends an `## Integration suite fix` section: the result, the pre-fix head, each fix commit, the cleared identities, and for any result but `fixed`, the remaining identities, the detail, and a way-forward sentence naming the pre-fix head and the fix commits so the operator decides whether to keep them (`git reset --hard <pre-fix head>` drops them).

In `landing-template-describe.md`, carry an `## Integration suite fix` section into the description as a reviewer check item naming the fix commits and the identities they cleared, beside the two existing integration sections.
In `webster-spec.md`'s summary section, describe the new section and when it is written.

Tests: `integrationreport_test.go` round-trips a report carrying `Fix` and parses one without it;
`summary_test.go` asserts a `fixed` section names the commits and cleared identities and carries no way-forward sentence, and a `failed` section names the pre-fix head, the commits, the remaining identities and the way forward.

**ImpactSummary:** the integration report schema gains one optional Go-written field and summary.md one optional section; the Describe stencil learns to carry it.
