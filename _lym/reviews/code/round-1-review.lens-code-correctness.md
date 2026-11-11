```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the codexengine sandbox-probe diff.

**Per-function list**

- `runSandboxProbe`, exit 0: returns nil. Correct. I ran `codex sandbox -c 'sandbox_mode="workspace-write"' -- true` with an existing `CODEX_HOME`. It exits 0, and Codex prints a harmless "PATH aliases" warning on stderr.
- `runSandboxProbe`, non-zero exit: returns an error holding the exit status and the trimmed output. Correct.
- `runSandboxProbe`, missing binary (`exec: not found`): returns an error with no output. Correct as an error, but it is classified as "sandbox cannot start". See F1.
- `runSandboxProbe`, 30 s timeout: `CommandContext` kills the process, so `err` is "signal: killed". Treated as "cannot start". See F1.
- `runSandboxProbe`, `CODEX_HOME` nonexistent: I ran this and it exits 1 with "CODEX_HOME points to ... does not exist". `readyHome` runs before the probe and creates the home, so this does not happen on the Prepare path. Correct.
- `runSandboxProbe`, `cmd.Dir=""` (engine from `New()` or empty `dirs.Anchor`): the probe runs in the process cwd. This does not fail. Correct, but the cwd is not the launch's cwd.
- `launchSandboxMode`, `danger-full-access`: returns full access with no probe. Correct.
- `launchSandboxMode`, `auto` with a passing probe: returns workspace-write. Correct.
- `launchSandboxMode`, `auto` with a failing probe: warns and returns full access. Correct for a real bwrap failure, but see F1 for transient causes. The warning goes only through `c.warn`, which is `logger.Warn`, so it never reaches the seat's state or the operator pane.
- `launchSandboxMode`, `workspace-write` with a failing probe: returns an error with the cause and the way forward. `Prepare` returns before it writes `prompt.md`. Correct, and the test checks that no prompt file exists.
- `Prepare` failure path: the home is already readied, but readying is idempotent. No inconsistent state.
- `OpenLaunch`: same call, same error return, before anything else is built. Correct.
- `LoadConfig`: `auto`, `workspace-write` and `danger-full-access` are accepted, and an empty or other value is refused. Correct.
- `TestPrepare_SandboxMode` warning assertion: operator precedence works out for every row. Correct.

**Findings**

- F1. MEDIUM, design. `internal/shuttleengine/codexengine/sandbox.go:56-60`.
  - Under `auto`, any probe error drops the sandbox. That includes a 30 s timeout under load, a missing or non-executable binary, and a killed or signalled probe.
  - Only a real sandbox refusal should degrade `auto`. A transient failure silently removes the write bound for that seat; for a missing binary the launch then fails anyway.
  - Fix: have `runSandboxProbe` tell a probe that ran and failed (`*exec.ExitError`) from one that could not run or timed out. Degrade only on the first. For the second, return an error or retry. The `ctx.Err()` check and the `exec.ErrNotFound` check go at `sandbox.go:30-37`.
- F2. LOW, design. `internal/shuttleengine/codexengine/sandbox.go:19-33`.
  - The probe does not run the policy the launch uses. It sets only `sandbox_mode`, with no `writable_roots` and no network-access setting.
  - The cwd is `c.dirs.Anchor`, which is empty for an engine from `New()` and falls back to the process cwd.
  - bwrap's namespace setup is the same either way, so the verdict is probably the same.
  - I did not confirm that `-c sandbox_mode=...` changes the `codex sandbox` policy; the exit-0 runs above show only that the flag is accepted. If it does not, the probe still checks the default policy, which is what matters.
  - Fix: either state this limit in the `runSandboxProbe` doc, or pass the same `-c` overrides the launch line carries.
- F3. LOW, design. `internal/shuttleengine/codexengine/sandbox.go:57-60`.
  - The `auto` fallback warning goes only to `logger.Warn`. The summary admits this ("reports only through lym's log"). The focus asks whether it reaches the operator; it does not, so a seat silently loses its sandbox.
  - Fix: either record the fallback in the launch's reported state, or accept it and word the doc and `template.yaml` to say it is log-only.

**Tool calls**
1. Bash: ran `codex sandbox` with three `CODEX_HOME` and `sandbox_mode` variants, to check the subcommand and flag shape and the exit codes. The context did not show real behaviour.
2. Read `codexengine.go:95-164`: to see `Prepare`'s ordering around `launchSandboxMode`, which the diff shows only as hunks.
