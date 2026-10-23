# Card 6 — fixer-report

**Create:**
- `contracts/specs/landing-fixer-report-spec.md`
- `plan:contracts/specs#LandingFixerReportSpec` -> `var LandingFixerReportSpec []byte`
- `plan:internal/landingshed#FixerVerdict` -> `type FixerVerdict string`
- `plan:internal/landingshed#FixerFixed` -> `const FixerFixed FixerVerdict = "fixed"`
- `plan:internal/landingshed#FixerEscalate` -> `const FixerEscalate FixerVerdict = "escalate"`
- `plan:internal/landingshed#FixerReport` -> `type FixerReport struct`
- `plan:internal/landingshed#ParseFixerReport` -> `func ParseFixerReport(path string) (FixerReport, error)`
- `plan:internal/landingshed#TestParseFixerReport` -> `func TestParseFixerReport(t *testing.T)`

**Extends:**
- `contracts/specs#entries`
- `pattern/PATTERN-sole-parsers.md`

**Intent:** Pin the merge fixer's report format in a new spec and give it one parser in `internal/landingshed` (D4, `PATTERN-sole-parsers`).
The spec `landing-fixer-report-spec.md` defines a markdown file whose first line is `verdict: fixed` or `verdict: escalate`, then a `## Parent changes` section with one bullet per parent change the fixer judged against the decision record and its finding on it (empty in `finalize` mode), a `## Fixes` section naming each commit the fixer made and what it fixed, and, for `escalate`, a `## Conflict` section naming the parent change and the decision of the record they conflict over.
It is embedded as `LandingFixerReportSpec` and registered in `entries` as `landing-fixer-report-spec`, so the hub deploys it beside the other specs.

`ParseFixerReport` reads the file and returns a `FixerReport` holding the verdict, the parent-change findings, the fixes and the conflict text.
A missing file is an error wrapping `fs.ErrNotExist`, so a caller tells a missing report from an unparseable one; a file whose first line is no known verdict, or an `escalate` report without a `## Conflict` section, is an error naming the line and the spec.
The parser is the only reader of the format, and the background file names it as the fixer report's sole parser.

The table-driven test pins a `fixed` report with parent changes and fixes, a `finalize`-mode `fixed` report with an empty parent-change section, an `escalate` report with its conflict, a missing file, an unknown verdict and an `escalate` report missing its conflict.

**ImpactSummary:** One registry line and one background-file section added; no existing spec or parser changes.
