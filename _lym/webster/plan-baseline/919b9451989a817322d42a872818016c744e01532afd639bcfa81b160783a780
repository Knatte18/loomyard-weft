# Card 11 — webster-runlevel-entry-report

**Create:**
- `plan:internal/websterengine#wrapRunLevelEntry` -> `func wrapRunLevelEntry(entry shuttleengine.GateEntry, reportPath, fixPromptPath string, notes *VerifyGateNotes) shuttleengine.GateEntry`
- `plan:internal/websterengine#TestWrapRunLevelEntry` -> `func TestWrapRunLevelEntry(t *testing.T)`

**Extends:**
- `internal/websterengine#VerifyGateReport`

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine#renderVerifyGateFindings`
- `internal/websterengine#TestRenderVerifyGateFindings`
- `internal/websterengine/doc.go`
- `contracts/stencils/webster/webster-body-verify-fix.md`

**Uses:**
- `internal/websterengine#WriteVerifyGateReport`
- `internal/websterengine#VerifyGateReportPath`
- `internal/websterengine#verifyGateReprompt`

**Intent:** Wire, under the operator's decision 3, how a failed `drift` entry at Merriam's run-level gate reaches a fixer (D8): through the same paths a failed `verify` entry takes, so Master still edits no code.

- `VerifyGateReport` gains `Entry` (the name of a run-level entry other than `verify` that failed) and `Findings` (that entry's findings text), both `omitempty`.
- `wrapRunLevelEntry` returns `entry` with a gate that, on a failed result, writes a `VerifyGateReport` carrying `Entry`, `Findings` and the attempt count to `reportPath` through `WriteVerifyGateReport` before returning the result, and with `verifyGateReprompt(fixPromptPath, notes)` as its `Reprompt`, so Master's re-prompt names the fixer fork or `lym webster fix-verify` and the verify-fix prompt file exactly as for `verify`.
  A passing result writes nothing.
- `Run` wraps every entry of `deps.Gate` with it before appending the `verify` entry.
- `renderVerifyGateFindings` renders an `Entry` report as the entry's name and findings, followed by the same way forward a verify failure gets.
- The `webster-body-verify-fix` stencil tells the fixer that a report may carry a named entry's findings, such as the `drift` entry's changed values outside the sealed expectation, and that it never edits `_lym/drift/expected.yaml`.
  A value the task's decision record or plan shows the change meant to move is intended: the fixer leaves it as it is and says so in its reply, so the entry stays failing, Webster stops, and the run's parent amends the expectation with `lym drift amend` at that Stuck (D11 admits the writer's Stuck); any other value it fixes in code like a failing test.

`TestWrapRunLevelEntry` (tier0, `t.TempDir`) covers: a failing entry writes a report naming the entry and its findings and its reprompt names the fix prompt; a passing entry writes no report.
`TestRenderVerifyGateFindings` gains a row for an `Entry` report.
Bound: the wrap changes how a failure is reported, never whether the entry passes.

**ImpactSummary:** One call site in Run wraps the recipe's run-level entries; the verify entry and its report path are unchanged; the report gains two optional fields its YAML round-trip keeps.
