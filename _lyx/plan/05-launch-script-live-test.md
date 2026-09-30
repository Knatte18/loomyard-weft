# Card 5 — reedengine: live tmux proof that the sourced script behaves as the typed line did

**Create:**
- `internal/reedengine/launchscript_integration_test.go`

**Uses:**
- `internal/reedengine#Engine.AddStrand`
- `internal/reedengine#Engine.Down`
- `internal/reedengine#executablePath`
- `internal/reedengine#composePaneLaunchLine`
- `plan:internal/reedengine#launchScriptPath`

**Intent:**
Prove against a real tmux that sourcing the launch script runs the same statements in the pane shell's own scope that typing them did: the command runs, and the prelude's exports persist in the pane session after it returns.
The unit tests in cards 2–4 pin strings and files; only a live pane shell can show the scope is right.

The file is tagged `//go:build integration && !windows`, since the decision covers the POSIX pane shell; it builds its engine with `newColdScratchEngine`, which self-skips when the multiplexer is absent and tears everything down.

One test, `TestLaunchScript_SourcedScriptRunsInThePaneShellScope`:

1. Inject `executablePath` via `withInjectedExecutablePath` to return `<t.TempDir()>/fakebin/lyx`, a path that is never executed, so the test re-execs nothing (Live-Substrate Spawn Observability).
2. `AddStrand` with a `Cmd` that writes a marker line to a file under `t.TempDir()`, paths quoted through `shell.Posix().Quote`.
3. Wait with `waitUntil` for the marker file to hold the marker: the strand command ran.
4. Assert the script at `launchScriptPath(shell.ForGOOS(), e.stateDir(), strand.GUID)` holds `composePaneLaunchLine(shell.ForGOOS(), cmd, strand.GUID) + "\n"`.
5. Type a probe into the strand's pane with `send-keys -l` then `Enter` that writes `"$LYX_BIN"` and `"${PATH%%:*}"` on two lines to a second file, and wait for it.
   Assert the first line equals the injected executable path and the second equals its directory: `LYX_BIN` stayed exported and the spawning binary's directory is first on `PATH` after the strand command returned.
6. `Down`, then assert `reed.json` and the launch directory are both gone.

The test reads the command's side effects, never pane text, so nothing matches pane output against the launch line's content.

**Commit:** 5: reedengine: live tmux proof for the sourced launch script

**Verify:** go test -tags integration -run TestLaunchScript ./internal/reedengine/
