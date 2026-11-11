# Codex sandbox: probe at launch, auto mode, refuse an unstartable seat

A Codex seat now runs on a machine whose bubblewrap cannot create a user namespace, with nothing set by hand.
Before, `codex.yaml`'s `sandbox_mode` defaulted to `workspace-write`, and on such a machine every shell command in the session failed with `bwrap: setting up uid map: Permission denied`.
The seat wrote no output and ended as `stopped`, with no error recorded.
This is the failure of the `astra` assist on OSL-1046 (Ubuntu 24.04, whose AppArmor restricts unprivileged user namespaces).

## What changes

- `sandbox_mode` takes a third value, `auto`, and the template now sets it.
- A launch under `auto` or `workspace-write` first probes Codex's sandbox: `codex sandbox -c sandbox_mode="workspace-write" -- true`, run under the lym Codex home in the worktree.
- Under `auto`:
  - A sandbox that starts runs `workspace-write`.
  - A sandbox that refuses to start runs `danger-full-access`, with a warning naming the cause (bubblewrap's output).
- Under an explicit `workspace-write`, a sandbox that refuses to start refuses the launch.
  The error names the cause and the way forward, `lym config codex --layer machine --set sandbox_mode=auto`.
  The seat reads as not started rather than stopped.
- `danger-full-access` runs no probe and behaves as before.
- Both `Prepare` and `OpenLaunch` settle the mode through the new `launchSandboxMode` in `codexengine/sandbox.go`.
  `Prepare` refuses before `prompt.md` is written.
- `docs/overview.md`, the package doc and the template comment describe the new modes.

## Design decisions

- `auto` falls back to no sandbox rather than failing, so a Codex seat runs like a Claude seat, which has no sandbox either.
  A machine that wants the bound regardless pins `workspace-write` in its machine layer.
- A probe that cannot reach a verdict refuses the launch under both `auto` and `workspace-write`.
  Such a probe is a missing or unexecutable binary, a timeout (30 seconds) or a signal.
  It shows nothing about the sandbox, so dropping the write bound on it would be unfounded.
  The error wraps `errSandboxProbeNotRun` and names the cause.
- The probe checks that the sandbox starts.
  It does not run the launch's writable roots or network setting.
- `sandbox_mode` stays a machine key: the machine layer may set it, the local layer may not.
- The probe is injected as `Codex.sandboxProbe`, so no unit test runs Codex.
  `TestRunSandboxProbe` runs the real probe against a fake executable.

## Not confirmed

The run worked on Neve and could not reproduce on OSL-1046.
Check by hand on OSL-1046:

- That AppArmor's user-namespace restriction is the cause there.
  The probe's warning under `auto` should name `bwrap: setting up uid map: Permission denied`.
- That a Codex seat launched there under the template's `auto` runs, with that warning in the log.
- That an `astra` seat on OSL-1046 completes work end to end, with no sandbox.

## Findings to check

- `OpenLaunch`'s refusal under `workspace-write` is pinned by `WorkspaceWriteSandboxCannotStart` in `open_test.go`.
  Check that this covers the refusal and its error text.
- A probe that fails to run no longer degrades `auto` to `danger-full-access`; it refuses the launch.
  Check this in `launchSandboxMode` (`codexengine/sandbox.go`) and its tests `AutoProbeDidNotRun` and `WorkspaceWriteProbeDidNotRun` in `prepare_test.go`.
