# Python guard hook reads deny_python live on each call

A hub that sets `deny_python: false` in its shuttle config could still have Python refused in running sessions.
The python guard's hook was baked into the session's settings at launch, so flipping the flag reached a session only after a restart of the session and its watchers.

The python guard row is now installed in every launch, for both the Claude and Codex engines.
On a matching call its hook asks the hub's current config, and it denies only when Python is not allowed.
The system-prompt notice still follows `deny_python` as it stood at launch.

## What changes

- New hidden verb `lym shuttle hook-python-allowed` (`shuttlecli/hookpython.go`), named by `shuttleengine.PythonAllowedHookVerb`.
  It exits 0 when `deny_python` is false and 1 when it is true.
  It prints nothing on standard output, and it skips the shared geometry and engine resolution of `lym shuttle`.
- The verb finds the config from the session's working directory.
  Under a worktree of a hub whose Bolt checkout holds `_lym`, it loads the hub's layers, as a launch's hub mode does.
  Otherwise it loads the state directory a standalone run derives for the repository root, when that directory exists.
  `cliwire.StandaloneStateDir` is added for this: it derives the directory without creating anything or redirecting a sink.
- `claudeengine` and `codexengine` install the python row unconditionally.
  The row's command is `! <check> && <deny>`, so a failing check keeps the deny.
  Each engine has a separate "announced" switch table for the notice, so the notice follows the launch-time value while the hook follows the live one.
- The hook runs the launching `lym` binary by absolute path, not whichever `lym` is on `PATH`.
  `claudeengine.LymBinary` resolves it in `Prepare` and refuses a test binary, as `selfexec.LymBinary` does.
  `Prepare` fails if the binary cannot be resolved.
- The `lym config` help text, `template.yaml`, `shellguard` docs, `shuttleengine.Config` and `docs/overview.md` describe the new behaviour.

## Design decisions

- The check fails closed.
  A missing or failing `lym`, a project directory the hook cannot enter, a config that cannot be found, or a config that does not load all exit non-zero, so the hook keeps denying.
- The hook is installed even when `deny_python` is false at launch.
  This is what lets a later change to true take effect without a restart as well.
- The Claude hook runs the check from `${CLAUDE_PROJECT_DIR:-.}`; the Codex hook runs it from its own working directory, the session's.
- No hook command or engine signature changes beyond the added `lymPath` argument to the settings and guard builders.

## Tests

- `internal/orchcli/denypython_integration_test.go`: an orch launch with `deny_python: false` at the default layer carries no notice, and its hook allows Python.
- `internal/shuttlecli/hookpython_test.go` and `hookpython_standalone_test.go` cover the verb in hub mode, in standalone mode, and on a config load failure.
- The `claudeengine` and `codexengine` tests, including the settings golden file, cover the unconditional row, the live check and the separate announcement.
- The `testmain_test.go` files across engines and CLIs point `claudeengine.LymBinary` at a stand-in, since a test binary is refused.

## To check by hand

- Not run: flip `deny_python` in a real hub with a running session open, and confirm that session's next `python` call is allowed without a restart, and that flipping it back denies again.
- Failure path of the loaded config in `hook-python-allowed`: confirm that a hub config that fails to load keeps the hook denying and prints only a JSON error on standard error.
- The hook's bare-`lym` resolution: confirm in a real launch that the settings carry the absolute path of the launching binary.
- Standalone fallback in `shuttleConfigDirs`: it applies only when the state directory already exists; confirm a standalone run that has never created one keeps denying.
