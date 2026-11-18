# Card 12 — triage-in-fixer-reports

**Create:**
- `plan:internal/weaverengine#TestFailureReport` -> `func TestFailureReport(t *testing.T)`
- `plan:internal/websterengine#triagedPackageLog` -> `var triagedPackageLog string`

**Edit:**
- `internal/loomshed#verifyFailureFindings`
- `internal/runplan#VerifyGateReport`
- `internal/runplan#RenderVerifyGateFindings`
- `internal/websterengine#newVerifyGate`
- `internal/weaverengine#failureReport`
- `internal/loomshed#TestVerifyFailureFindings`
- `internal/websterengine#TestRenderVerifyGateFindings`
- `internal/websterengine#TestVerifyGate_FailedEvaluationWritesReport`

**Extends:**
- `internal/loomshed/doc.go`
- `internal/runplan/doc.go`
- `internal/websterengine/doc.go`
- `internal/weaverengine/doc.go`

**Uses:**
- `plan:internal/gateslot#TriageLines`

**Intent:**
Make every fixer's report carry the triage lines of its failed tests (D7's last bullet).
The card fixer's report is the call's stderr, which holds the failure report, so it carries them already (`detachRun.fixAndRerun`).
The other reports re-render failures from the raw test output, so they drop the failure report's four-space lines; each now adds them from the verify log through `gateslot.TriageLines`:

- `loomshed.verifyFailureFindings`, the findings Darn-Write's cold fixer and the writers of the loomshed verify gate read, gains a `Triage:` section after `Failures:` listing each failed test's triage lines under its identity, when the log holds any.
- `runplan.VerifyGateReport` gains `Triage map[string][]string` (yaml `triage`, omitted when empty); `RenderVerifyGateFindings` renders it as the same `Triage:` section.
  `websterengine.newVerifyGate` sets it from the failed run's log, for the first run and the rerun, so the webster fix-verify fixer's report carries it.
  `weaverengine.failureReport` sets it from the batch gate's log output.
- A log with no triage lines renders no section, so every existing report keeps its text.

Tests (tier0):
- `TestVerifyFailureFindings` gains a row whose log holds a failure report with triage lines: the findings hold the `Triage:` section with them; an existing row without triage lines keeps its findings.
- `TestRenderVerifyGateFindings` gains a row with `Triage` set rendering the section.
- `TestVerifyGate_FailedEvaluationWritesReport` gains rows over `triagedPackageLog`, a test log beside `failingPackageLog` holding the raw failing output followed by a failure report with a triage line under the failed test:
  the written report's `Triage` holds that line for a first run that fails, and for a rerun that fails again.
- `TestFailureReport` is one table over `weaverengine.failureReport`: an output with triage lines fills `Triage`; a timed-out run and an output without them leave it empty.

Each package's `doc.go` gains the triage section in its findings or report passage.

**ImpactSummary:** Three report renderers gain an optional section filled from one gateslot reader; the section is absent when the log has no triage lines, so existing findings text and the tests asserting it are unchanged.
