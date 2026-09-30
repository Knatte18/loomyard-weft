# Card 3 — verify-capture

**Edit:**
- `integration.go`
- `runverify_test.go`

**Create:**
- `plan:internal/websterengine#verifyRun` -> `type verifyRun struct`
- `plan:internal/websterengine#verifyRunner` -> `type verifyRunner func(verifyCmd, worktree, logPath string) (verifyRun, error)`
- `plan:internal/websterengine#runVerifyCapture` -> `func runVerifyCapture(verifyCmd, worktree, logPath string) (verifyRun, error)`
- `plan:internal/websterengine#IntegrationLogPath` -> `func IntegrationLogPath(scratchDir string) string`

**Intent:** Replace the output-discarding `runVerifyCommand` with a capturing variant, since triage and the identity-based bisect both need the verify command's output.

`verifyRun` holds `Passed bool` and `Output string`.
`runVerifyCapture` keeps `runVerifyCommand`'s shell selection (`sh -c`, `cmd /C` on Windows), its spawn/exit log lines, and its error contract (a non-zero exit is `Passed: false` with a nil error; a spawn failure is a real error).
It captures combined stdout and stderr into `Output`, and when `logPath` is non-empty it also writes that output to `logPath`, creating the parent directory;
a failed log write is a returned error, since a triage decision must never rest on a log that silently was not written.
`verifyRunner` is the function type triage and bisect take as their seam, so their untagged tests can inject a fake;
`runVerifyCapture` satisfies it.
Remove `runVerifyCommand` and switch `checkoutAndVerify` to `runVerifyCapture(verifyCmd, worktree, "")`, reading `.Passed`, so bisect behaves exactly as before in this card.

`IntegrationLogPath` returns the integration fork's verify log path, `<scratchDir>/verify/integration.log`.
Beside it, add unexported helpers for the triage rerun and baseline logs in the same directory (`rerun.log`, `baseline.log`).
All three live under webster's `.lyx` scratch dir, never under `_lyx`, per the Durable-vs-Ephemeral State Invariant.

In the `integration`-tagged `runverify_test.go`, retarget the existing cases at `runVerifyCapture` and add:
a failing command whose `Output` holds both its stdout and stderr lines and whose log file holds the same bytes;
a passing command with an empty `logPath` writing no file.

**ImpactSummary:** `runVerifyCommand` has one caller, `checkoutAndVerify`, retargeted in this card; bisect's pass/fail behavior is unchanged.

**Commit:** 3: verify-capture
