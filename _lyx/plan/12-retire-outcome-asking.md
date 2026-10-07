# Card 12 — retire-outcome-asking

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
- `internal/mergeresolve/mergeresolve.go`
- `internal/frictionengine/reflect.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/doc.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/webstercli/run.go`
- `internal/webstercli/ownership.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `internal/shuttleengine/run_test.go`

**Prosa:**
- `README.md`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`
- `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`

**Uses:**
- `internal/shuttleengine#MessageTail`
- `internal/shuttleengine#Runner.SetNotifier`

**Intent:**
D4 and D7, the last step: `OutcomeAsking` retires as an outcome, now that no caller references it.

- `engine.go`: delete the `OutcomeAsking` constant and its doc; the `Outcome` doc names done, died and timeout.
- `attach.go`: the legacy record value `"asking"` that `dispositionCandidate`, `reconstructAndWait` and `isTerminalOutcome` still read becomes one unexported constant documented as a value only an older binary wrote; `isTerminalOutcome` keeps treating it as terminal for a dead or untracked strand.
- `rundir.go`: `RunState.Outcome`'s doc lists the values finalize writes now and the legacy one.
- `shuttlecli`: `lyx shuttle run`'s help text and the group's `Long` say a run prints `done`, `died` or `timeout`, that a turn end without the output files holds the run until the outputs exist, the pane dies or the deadline passes, and that an interactive run is answered in its pane; `runCmd`'s doc comment follows.
- `shuttleengine/doc.go`: no remaining mention of asking as an outcome.
- A grep for `OutcomeAsking`, `AwaitOperator`, `ErrMasterAsking`, `MasterAskingError` and `runTriage` over the repository returns nothing outside `_lyx/` and `docs/benchmarks/` (a measurement report, left as recorded).
- Prose that still names asking as a shuttle run outcome goes too, since the symbol grep cannot see it: the `asking/died/timeout` outcome lists in the comments of `burlerengine/engine.go`, `burlerengine/doc.go`, `burlerengine/smoke_cluster_test.go`, `frictionengine/reflect.go` and `mergeresolve/mergeresolve.go` lose `asking`;
  the `lyx webster run` help text in `webstercli/run.go` says the spawn ends died or timed-out;
  `webstercli/ownership.go`'s "shuttle-asking exit" comment, `loomcli/smoke_attachprobe_test.go`'s "classify the run asking" comment and `shuttleengine/run_test.go`'s "kept asking/died/timeout dir" comments follow the three outcomes.
  After this card every remaining use of the word `asking` in Go comments, help text and docs is unrelated to a shuttle run outcome, apart from `DeadReasonAsking` and `docs/benchmarks/`.

Tests: switch the remaining uses of `OutcomeAsking` as a generic non-done outcome in `burlerengine/engine_test.go`, `frictionengine/reflect_test.go` and `mergeresolve/mergeresolve_test.go` to `OutcomeDied` or delete the row where it duplicates one, and clear any left in `attach_test.go` and `wait_test.go`.
The `llm`-tier smoke tests must keep compiling under `go vet -tags llm ./...`:
`TestSmokeGuardrailAskingSurfacesQuestion` becomes the question case of the new reading: an autonomous run whose agent states a question and ends its turn without the output file keeps running (its events file holds the Stop while `Wait` has not returned), the test answers through `Run.Send` with a text ending in `MessageTail`, and the run then finishes `done` with the file written;
it sets a recording notifier through `SetNotifier` and asserts exactly one call for the held turn end, which makes it the live scenario for the hold's notice (card 16 adds none of its own for it);
`smoke_interrupt_test.go` accepts only `done` after its redirect, its comments describing the interrupted turn's Stop as a hold.

Docs: `README.md`'s shuttle sentence and `docs/overview.md`'s shuttle module entry describe the three outcomes, the hold with its parent notice, and the payload-first background reading;
`SANDBOX-SHUTTLE-SUITE.md`'s asking scenario becomes a hold scenario (the run keeps waiting, the operator answers in the pane or by `lyx shuttle send`, the run ends `done`), and its interrupt note drops the `asking` envelope;
`SANDBOX-WEBSTER-SUITE.md`'s Master-grounding note drops the asking classification.

**ImpactSummary:** Removes the exported OutcomeAsking constant, which no caller references after cards 9–11; `lyx shuttle run` never prints asking, and the user-facing docs, help text and comments that named it follow.
