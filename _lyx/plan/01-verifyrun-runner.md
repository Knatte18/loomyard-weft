# Card 1 — verifyrun: the shared shell-command runner

**Create:**
- `plan:internal/verifyrun#Run` -> `func Run(ctx context.Context, command, dir string, out io.Writer) (int, error)`
- `internal/verifyrun/run_integration_test.go`

**Prosa:**
- `docs/overview.md`

**Intent:**
Create the new leaf package `internal/verifyrun` (source file `internal/verifyrun/run.go`, with a package doc comment) holding the one in-process shell-command runner that webster's integration bisect and landing's post-merge verify gate both call, so shell selection lives in one place.
`internal/shell` cannot host it: the Shell Mechanics Seam keeps that package stdlib-only, and this runner logs through `internal/logger`.
The package imports only the standard library and `internal/logger`.

`Run` runs `command` through the platform shell — `sh -c` on POSIX, `cmd /C` when `runtime.GOOS == "windows"` — built with `exec.CommandContext(ctx, ...)`, with `cmd.Dir = dir` and both `cmd.Stdout` and `cmd.Stderr` set to `out`, so combined output lands in the caller's writer.
When `out` is `io.Discard`, leave both nil instead, so the child writes to the null device directly:
`os/exec` special-cases only nil and `*os.File`, and would otherwise open a copy pipe that a lingering child can hold open until the wait delay expires, a teardown webster's bisect never had.
It sets `cmd.WaitDelay` to a package constant (10 seconds, mirroring `internal/treadleengine`'s `gateWaitDelay`) so a cancelled context kills the shell and the call cannot hang on a child still holding the output pipe.
Return contract:
- exit 0 → `(0, nil)`;
- a non-zero exit (`*exec.ExitError`) with `ctx` not cancelled → `(exitCode, nil)`;
- `ctx` cancelled (checked via `ctx.Err()` after the wait returns, before classifying the error) → `(-1, err)` where `err` wraps `ctx.Err()` so `errors.Is(err, context.Canceled)` holds;
- `exec.ErrWaitDelay` after a completed process → classify by `cmd.ProcessState` exactly as `treadleengine`'s gate does;
- any other error (the shell could not start) → `(-1, err)` wrapping the cause and naming the command.

Keep today's webster spawn and teardown log lines, per the Live-Substrate Spawn Observability rule: a `logger.Info` "spawning verify command" with `shell`, `command` and `dir` before the run, a `logger.Info` "verify command exited" carrying an `exitCode` key on every exit (0 or non-zero), and a `logger.Warn` "verify command failed to spawn" carrying a `cause` key on a spawn failure.
A cancellation logs an `Info` teardown line naming the cancellation, never a `Warn`.

The integration-tagged test (it spawns real processes, per the Test Tier Purity Invariant) covers: a passing command returns `(0, nil)` and its stdout and stderr both reach `out`; `exit 3` returns `(3, nil)`, both with a buffer and with `io.Discard` as `out`; the working directory is `dir` (a command printing its working directory writes `dir`); a cleared `PATH` makes the shell unresolvable and returns a non-nil error with a `WARN` log line carrying `cause`; a context cancelled while a long-running command (`sleep 30` on POSIX, a `ping -n 30 127.0.0.1` on Windows) runs returns an error satisfying `errors.Is(err, context.Canceled)` well before the command's own duration.
Capture log output the way `internal/websterengine/runverify_test.go`'s `runverifyCapture` does.
The package needs no `TestMain`: its tests spawn no git, so the Hermetic Git Test Environment Invariant does not bind it, and they carry none of `cmd/lyx`'s git-spawn scan tokens.

Add `internal/verifyrun/` to the package tree in `docs/overview.md`, beside `internal/shell/`, described as the in-process shell-command runner shared by webster's integration bisect and landing's post-merge verify gate.
Also add `internal/verifyrun` to the shared-infrastructure sentence's package list ("The user-facing modules sit on a thin layer of shared infrastructure (…)"), after `internal/shell`.

**Commit:** 1: verifyrun: add the shared in-process shell-command runner
