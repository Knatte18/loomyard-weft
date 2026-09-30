# Card 2 — webster: run the bisect verify through the shared runner

**Edit:**
- `internal/websterengine#runVerifyCommand`

**Uses:**
- `plan:internal/verifyrun#Run`

**Intent:**
Replace `runVerifyCommand`'s own shell selection and `exec.Command` call with a call to `verifyrun.Run(context.Background(), verifyCmd, worktree, io.Discard)`, so webster and landing share one runner instead of two drifting copies.
Webster's behaviour stays as it is today: a background context (the bisect has no cancellation to honour) and a discarding writer (the bisect never captured output), which `verifyrun.Run` maps to the null device exactly as today's nil `cmd.Stdout` did.
Map the result back onto the existing `(bool, error)` contract: a returned error is wrapped as today (`webster: bisect: run verify command %q: %w`), and otherwise `passed` is `exitCode == 0`.
Drop the now-unused `os/exec`, `runtime` and `logger` imports from `integration.go` if nothing else in the file uses them; the spawn and teardown log lines now come from `verifyrun.Run`.
Update `runVerifyCommand`'s doc comment to say it delegates to the shared runner.

The existing integration-tagged `TestRunVerifyCommand` in `internal/websterengine/runverify_test.go` already asserts the non-zero-exit and spawn-failure paths and their log lines, and must pass unchanged — it is this card's regression test.

**ImpactSummary:** `runVerifyCommand` keeps its signature; its only callers are webster's bisect and its own test, and both see identical pass/fail and log behaviour.

**Commit:** 2: websterengine: run the bisect verify through the shared runner
