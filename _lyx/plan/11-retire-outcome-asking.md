# Card 11 — retire-outcome-asking

**Edit:**
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/attach.go`
- `internal/shuttleengine/rundir.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/attach_test.go`
- `internal/shuttleengine/wait_test.go`
- `internal/shuttlecli/run.go`
- `internal/shuttlecli/cli.go`
- `internal/shuttlecli/smoke_guardrail_test.go`
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/burlerengine/engine_test.go`
- `internal/frictionengine/reflect_test.go`
- `internal/mergeresolve/mergeresolve_test.go`

**Prosa:**
- `README.md`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`
- `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`

**Uses:**
- `plan:internal/shuttleengine#MessageTail`

**Intent:**
D4 and D7, the last step: `OutcomeAsking` retires as an outcome, now that no caller references it.

- `engine.go`: delete the `OutcomeAsking` constant and its doc; the `Outcome` doc names done, died and timeout.
- `attach.go`: the legacy record value `"asking"` that `dispositionCandidate`, `reconstructAndWait` and `isTerminalOutcome` still read becomes one unexported constant documented as a value only an older binary wrote; `isTerminalOutcome` keeps treating it as terminal for a dead or untracked strand.
- `rundir.go`: `RunState.Outcome`'s doc lists the values finalize writes now and the legacy one.
- `shuttlecli`: `lyx shuttle run`'s help text and the group's `Long` say a run prints `done`, `died` or `timeout`, that a turn end without the output files holds the run until the outputs exist, the pane dies or the deadline passes, and that an interactive run is answered in its pane; `runCmd`'s doc comment follows.
- `shuttleengine/doc.go`: no remaining mention of asking as an outcome.
- A grep for `OutcomeAsking`, `AwaitOperator`, `ErrMasterAsking`, `MasterAskingError` and `runTriage` over the repository returns nothing outside `_lyx/` and `docs/benchmarks/` (a measurement report, left as recorded).

Tests: switch the remaining uses of `OutcomeAsking` as a generic non-done outcome in `burlerengine/engine_test.go`, `frictionengine/reflect_test.go` and `mergeresolve/mergeresolve_test.go` to `OutcomeDied` or delete the row where it duplicates one, and clear any left in `attach_test.go` and `wait_test.go`.
The `llm`-tier smoke tests must keep compiling under `go vet -tags llm ./...`:
`TestSmokeGuardrailAskingSurfacesQuestion` becomes the question case of the new reading: an autonomous run whose agent states a question and ends its turn without the output file keeps running (its events file holds the Stop while `Wait` has not returned), the test answers through `Run.Send` with a text ending in `MessageTail`, and the run then finishes `done` with the file written;
`smoke_interrupt_test.go` accepts only `done` after its redirect, its comments describing the interrupted turn's Stop as a hold.

Docs: `README.md`'s shuttle sentence and `docs/overview.md`'s shuttle module entry describe the three outcomes, the hold with its parent notice, and the payload-first background reading;
`SANDBOX-SHUTTLE-SUITE.md`'s asking scenario becomes a hold scenario (the run keeps waiting, the operator answers in the pane or by `lyx shuttle send`, the run ends `done`), and its interrupt note drops the `asking` envelope;
`SANDBOX-WEBSTER-SUITE.md`'s Master-grounding note drops the asking classification.

**ImpactSummary:** Removes the exported OutcomeAsking constant; `lyx shuttle run` never prints asking; user-facing docs follow.
