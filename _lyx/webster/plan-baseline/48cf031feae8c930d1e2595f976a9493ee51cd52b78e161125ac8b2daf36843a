# Card 5 — burler-run-halves

**Edit:**
- `internal/burlercli#burlerCLI`
- `internal/burlercli#burlerCLI.runCmd`
- `internal/burlercli#resultEnvelope` -> `func resultEnvelope(review burlerengine.ReviewResult, fix burlerengine.FixResult, mode, stateDir, stencilsDir string) map[string]any`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/burlercli#Command`
- `internal/burlercli#TestResultEnvelope_ForkCountNilGuard`
- `internal/burlercli#TestWire_SelectsModeAndWiresItsFields`
- `internal/burlercli#TestWireStandalone_RunnerReachesPublicEntryPointWithoutToldPathError`
- `internal/burlercli#TestRunCmd`
- `internal/burlercli/doc.go`
- `tools/sandbox/SANDBOX-BURLER-SUITE.md`

**Intent:**
`lyx burler run` runs the reviewer and then, only when the review has findings, the fixer, over card 2's halves instead of the two-seat round.

`runCmd` decodes the profile as today, calls `Engine.Review`, and stops at a review that did not reach done, printing the envelope with the review's outcome.
For an accepted review it calls `Engine.Fix` with the review's digest and the same options; the CLI tells no gate, so a review with no findings is the engine's skip: no fixer session, a Go-written fixer report, and `fixSkipped` true in the envelope.
An error from either half is the error envelope, as today's `Engine.Run` error is.
The ready marker goes: `runCmd` derives none, `burlerCLI` loses `markerRoot` and `markerBase`, `wireHub` and `wireStandalone` stop filling them, and the long help drops the marker and the review-path-under-the-root refusal that came with it.
`resultEnvelope` takes both halves' results: `outcome` is the fix half's when it ran and the review's otherwise, `verdict` and `forkCount` and `clusterWarnings` come from the review, `review` and `fix` are the two halves (`halfEnvelope`), `fixSkipped` reports the skip, and `readyMarkerPath` is gone.
`Command`'s and `runCmd`'s help and `internal/burlercli/doc.go` describe the two sequential sessions, under the roles `burler-review-chair` and `burler-fix-chair`.

Tests: `TestResultEnvelope_ForkCountNilGuard` covers a review that stopped short, a fixed round and a skipped fix; `TestWire_SelectsModeAndWiresItsFields` drops the marker assertions; `TestWireStandalone_RunnerReachesPublicEntryPointWithoutToldPathError` drives `Engine.Review`; `TestRunCmd` covers a review with findings running the fixer, a review with none skipping it, and a died reviewer running no fixer.
`SANDBOX-BURLER-SUITE.md`'s S2 says the fixer report of a review with no findings is the Go-written one.

**ImpactSummary:** `lyx burler run`'s envelope loses `readyMarkerPath` and gains `fixSkipped`; nothing else calls the CLI's round, and the engine's old `Run` loses its last production caller.

**Commit:** 5: burler: lyx burler run runs the reviewer, then the fixer when there are findings
