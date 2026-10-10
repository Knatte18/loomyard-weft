# Card 4 — verify-gate-way-forward

**Create:**
- `plan:internal/websterengine#verifyGateWayForward` -> `func verifyGateWayForward(fixPromptPath string) string`
- `plan:internal/websterengine#verifyGateReprompt` -> `func verifyGateReprompt(fixPromptPath string) func(findingsPath string) string`

**Edit:**
- `internal/websterengine#NewVerifyGate` -> `func NewVerifyGate(geom Geometry, attempts int, batches []batcher.Batch, parentBranch ParentBranchFunc, frictionDir, fixPromptPath string) (shuttleengine.Gate, *VerifyGateNotes)`
- `internal/websterengine#newVerifyGate` -> `func newVerifyGate(reportsDir string, attempts int, notes *VerifyGateNotes, fixPromptPath string, s verifyGateSeams) shuttleengine.Gate`
- `internal/websterengine#renderVerifyGateFindings` -> `func renderVerifyGateFindings(r VerifyGateReport, fixPromptPath string) string`
- `internal/websterengine#Run`
- `internal/websterengine/doc.go`
- `internal/websterengine/verifygate_test.go`
- `internal/websterengine/verifygatereport_test.go`
- `internal/websterengine/verifygate_integration_test.go`
- `internal/websterengine/runlevel_git_test.go`
- `internal/webstercli/verify_parity_integration_test.go`

**Uses:**
- `internal/shuttleengine#GateEntry`

**Intent:** Make the verify gate name Master's way forward in both the re-prompt line and the findings file (decision D2).
`verifyGateWayForward` returns one line: do not fix the findings yourself; spawn one fixer fork as the gate-failure section of your prompt says, with the prompt file at `<fixPromptPath>`; then end your turn.
`verifyGateReprompt` returns the renderer `Run` puts on the verify entry: `Gate findings recorded at <findingsPath> — <way forward>`, one line, keeping the `Gate findings recorded at` opening the Master stencil's gate-failure section triggers on.
`NewVerifyGate` takes the verify-fix prompt's absolute path as `fixPromptPath` and hands it to `newVerifyGate`; `Run` passes the `verifyFixPromptPath` it already resolves and writes before building the gate, so no new error path arises.
`renderVerifyGateFindings` opens with the same way-forward line before the "Verify gate failed (attempt N of M)" line, on every failure shape.
`Run` sets `Reprompt: verifyGateReprompt(path)` on the `verify` gate entry it appends.
The Master stencil's gate-failure section is unchanged.
Update the "The verify-gate fixer fork" section of `doc.go`: the gate's re-prompt and its findings both open with the way forward, so the rule stands where Master reads the failure.

Tests: add assertions to `TestRenderVerifyGateFindings` that every rendered shape opens with the way-forward line naming the fixer-fork step and the prompt path;
add a case to `TestVerifyGate_FailedEvaluationWritesReport` (or the closest existing test over `newVerifyGate`) asserting the returned findings open with the way forward;
pin the wiring at `Run`'s surface in `TestRun_VerifyGate` (`runlevel_git_test.go`): `verifyGateOf(t, fx).Reprompt` is non-nil, and its line for a findings path is one line, opens with `Gate findings recorded at ` and that path, and names the verify-fix prompt path `Run` wrote.
Update the `newVerifyGate` calls in `verifygate_test.go`, and the `NewVerifyGate` calls in `verifygate_integration_test.go` and `verify_parity_integration_test.go`, for the new parameter.

**ImpactSummary:** One exported and two unexported re-signs whose callers sit in verifygate.go, runlevel.go and four test files across websterengine and webstercli, one new field set on the verify entry in Run; the gate's verdicts are unchanged.
