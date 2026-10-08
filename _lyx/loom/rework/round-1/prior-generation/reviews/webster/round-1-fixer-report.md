# Webster round 1 — fixer report

Every finding's premise held against the code and the plan, and all three are fixed, one commit each.

## Fixed

- **F1** (BLOCKING, `1164d422f`): `TestRun_Send_ConfirmsSubmission` and `TestStartup_SendFailuresTearDown` now call `t.Parallel()` at the top level and in each `t.Run` body.
  Each row builds its own fixture, fake clock and reed, and neither test uses `stubInputSleep` or `logcapture`, so no global state excuses a serial run.
  `go test -race -count=1 ./internal/shuttleengine/` passes.
- **F2** (LOW, `321a96f10`): card 6's Bound and D6 bound the unsent re-prompt's retry by the run deadline alone, so `maxUnsentReprompts` is gone.
  The `unsentReprompts` count becomes the plan's `unsentReprompt` mark (a bool) in `Run`, `Wait`, `handleGatedBoundary` and `boundaryIdle`, and the `wait.go` file doc and `handleGatedBoundary`'s doc comment name the deadline as the only bound.
  `TestGate_UnsentRepromptRetries`' row that asserted the cap now asserts the opposite: a re-prompt that stays busy is retried until the rig's 30 s run deadline (`wantEndAtLeast`).
  The old row passed while asserting the loop ended before 15 s, so the corrected assertion would fail against the capped code (inferred from that, not re-run).
  This corrects an existing row and adds no test.
- **F3** (LOW, `a37a862e9`): `send.go`'s file header now names the settle, submit confirmation and clear of an input-box send and the gated wait loop's send and boundary-idle checks, alongside the send context and the idle wait.
  I widened the header rather than moving the gate methods, which kept the diff small.

Verification after the last fix: `go build ./...`, `go vet` under the `integration`, `tmux` and `llm` tags, `go test -count=1 ./...` and `lyx loom lint-comments` all pass.

## Disputed

None.

## Deferred

None.
