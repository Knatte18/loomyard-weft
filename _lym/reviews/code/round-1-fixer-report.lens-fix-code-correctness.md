```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code (`runSandboxProbe`, `launchSandboxMode` and the added test rows) through the fix-code-correctness lens.

Changed-signature callers: `runSandboxProbe` keeps its signature. It is reached only through the `sandboxProbe` field, called from `launchSandboxMode`, which `Prepare` and `OpenLaunch` call. Nothing is left broken.

Findings:

1. **LOW**: the `exitErr.ExitCode() < 0` and `ctx.Err() != nil` branches of the `runSandboxProbe` classifier (`sandbox.go`, the `errSandboxProbeNotRun` return) have no test.
   - Evidence: `TestRunSandboxProbe` adds only the `ExecutableMissing` row, which takes the `!errors.As(err, &exitErr)` branch.
   - Removing `|| exitErr.ExitCode() < 0` fails no test. The fake-script harness could cover a signalled probe, e.g. a script ending in `kill -9 $$`.
   - The review itself rejected the timeout test because the timeout is not injectable, so this is only the signal case.

2. **LOW**: the `wantErr` on the `ExecutableMissing` row is dead data.
   - Evidence: `sandbox_test.go` returns inside the `if tt.missingBin` block, so the row's `wantErr` loop never runs. The row asserts through `errors.Is` instead.
   - It is harmless but misleading.

Traces that held:
- **Exit 1:** the error is an `*exec.ExitError` with code 1 and the context is live, so the plain wrapped error is returned. `errors.Is(err, errSandboxProbeNotRun)` is false, and `launchSandboxMode` degrades under auto or refuses under workspace-write, as before.
- **Missing binary or bad dir:** `Start` returns a non-`ExitError`, so the error wraps the sentinel. `launchSandboxMode` refuses under both auto and workspace-write, and the message carries `err`, so the cause appears.
- **Timeout kill:** `ctx.Err()` is non-nil, so the error wraps the sentinel.
- **Two `%w` in one `Errorf`:** this is valid in Go 1.20 and later, and `go vet` passed.
- **New `TestPrepare_SandboxMode` rows:** both `*ProbeDidNotRun` rows fail with the old code. Auto would degrade and return no error, so the `Fatal` fires. Both also assert the probe arguments.
- **New `TestOpenLaunch` row:** `WorkspaceWriteSandboxCannotStart` fails if the `OpenLaunch` error return is removed, because `err == nil` hits `Fatal`. It also asserts the probe arguments.
- **`!tt.wantProbe` in the home-unmade check:** this is correct, since the probe needs the lym home and the refusal comes after it is made.

Outside my lens, one line: the `runSandboxProbe` doc line "When the executable exits non-zero it returns an error carrying the command's output, such as bubblewrap's refusal to set up its namespace." is about 140 characters, over the 120-character prose limit.

Tool calls: none; my context held every changed file and the test output.
