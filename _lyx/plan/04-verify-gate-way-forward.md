# Card 4 — verify-gate-way-forward

**Create:**
- `plan:internal/websterengine#verifyGateWayForward` -> `func verifyGateWayForward(fixPromptPath string) string`
- `plan:internal/websterengine#verifyGateReprompt` -> `func verifyGateReprompt(fixPromptPath string) func(findingsPath string) string`
- `plan:internal/websterengine#verifyFixPromptFile` -> `func verifyFixPromptFile(geom Geometry) (string, error)`

**Edit:**
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#newVerifyGate` -> `func newVerifyGate(reportsDir string, attempts int, notes *VerifyGateNotes, fixPromptPath string, s verifyGateSeams) shuttleengine.Gate`
- `internal/websterengine#renderVerifyGateFindings` -> `func renderVerifyGateFindings(r VerifyGateReport, fixPromptPath string) string`
- `internal/websterengine#Run`
- `internal/websterengine/doc.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/verifygatereport_test.go`

**Uses:**
- `internal/shuttleengine#GateEntry`

**Intent:** Make the verify gate name Master's way forward in both the re-prompt line and the findings file (decision D2).
`verifyGateWayForward` returns one line: do not fix the findings yourself; spawn one fixer fork as the gate-failure section of your prompt says, with the prompt file at `<fixPromptPath>`; then end your turn.
`verifyGateReprompt` returns the renderer `Run` puts on the verify entry: `Gate findings recorded at <findingsPath> — <way forward>`, one line, keeping the `Gate findings recorded at` opening the Master stencil's gate-failure section triggers on.
`verifyFixPromptFile` returns the absolute path of the verify-fix prompt under `geom.PromptsDir`, the computation `Run` makes inline today; `Run` calls it instead, and `NewVerifyGate` calls it to hand `newVerifyGate` the path.
`renderVerifyGateFindings` opens with the same way-forward line before the "Verify gate failed (attempt N of M)" line, on every failure shape.
`Run` sets `Reprompt: verifyGateReprompt(path)` on the `verify` gate entry it appends.
The Master stencil's gate-failure section is unchanged.
Update the "The verify-gate fixer fork" section of `doc.go`: the gate's re-prompt and its findings both open with the way forward, so the rule stands where Master reads the failure.

Tests: add assertions to `TestRenderVerifyGateFindings` that every rendered shape opens with the way-forward line naming the fixer-fork step and the prompt path;
add a case to `TestVerifyGate_FailedEvaluationWritesReport` (or the closest existing test over `newVerifyGate`) asserting the returned findings open with the way forward;
pin `verifyGateReprompt` in `verifygate_test.go` as a row of an existing test: one line, opening with `Gate findings recorded at ` and the findings path, naming the prompt path.
Update the `newVerifyGate` calls in `verifygate_test.go` for the new parameter.

**ImpactSummary:** Two unexported re-signs with callers in verifygate.go and two test files, one new field set on the verify entry in Run; the gate's verdicts are unchanged.
