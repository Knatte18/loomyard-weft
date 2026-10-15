# reed: --focus=false keeps the viewer, launch line survives stray input

Fixes #627 and #628.

## What changes

`lyx shuttle run --focus=false` now never moves the viewer to the run's new window.
Before, `Focus: false` only meant "not forced": reed's follow-the-run rule (`newAgentWindowSelected`) still selected the new window whenever the viewer sat on the relevant window, the overview, a dead agent window or a driver window.
In the prime, the orch's own window is the relevant window, so each `--focus=false` run pulled the operator off the orch pane.

A new `render.Display.KeepViewer` flag (`keepViewer` in `reed.json`, omitted when false) turns that rule off for one strand.
`lyx shuttle run` sets it to the negation of `--focus`.
`Focus` still wins when both are set, and strands that set neither keep following the run as before, which is what loom's driver, arbiter and seat spawns rely on.
A window that keeps the viewer gets `@lyx_unseen` like any other agent window that is not selected.

`launchStrandLocked` now sends the shell dialect's discard-line key before it types the launch script's source statement.
The traces of the failed parallel launch show the runs serialized about 100 ms apart under reed's lock, each opening a window that the follow rule then selected.
So the viewer moved onto each pane while its shell was still booting, and input sent to the viewer's active pane (keystrokes or a terminal reply) landed in front of the typed line, giving `5. '<script>'`.
`C-u` discards that pending line both in the tty's canonical mode, before readline starts, and in readline itself.

`shell.Shell` gains `DiscardLineKey`: `C-u` on POSIX, and empty on pwsh.
When the key is empty, the launch types only the payload and `Enter`, as before.

## Design decisions

- Rather than changing what `Focus: false` means, the change adds a separate flag, because the zero value is the follow-the-run behavior internal spawners depend on.
- pwsh gets no discard key because PSReadLine's key for that is Escape, and an Escape sent right before the typed `.` reads as Alt+`.` (YankLastArg).
- `lyx reed add --focus` is unchanged. Its default is false and keeps meaning "follow".
- Two items the issues mention are left out because the fix doesn't need them: a flag to open a run's window outside the caller's session, and the `--anchor hidden` startup failure.

## Remaining risk

A window that does take the viewer, through `Focus` or the follow rule, can still receive stray input after the `C-u` and before the payload. That window is a single tmux round trip.

## Tests

- `TestNewAgentWindowSelected` adds cases where a keep-viewer strand moves no viewer from any window that would otherwise pass it on, plus one where `Focus` wins over `KeepViewer`.
- `TestRunCmd_EffortAndFocusFlags` (renamed from `TestRunCmd_EffortFlag`) pins the `Spec.Display` that `--focus` and `--focus=false` build.
- `TestLaunchStrandLocked_LaunchScript` pins the sequence: discard key (as a key, not literal text), then the literal source statement, then `Enter`.
- `TestShell_ScriptExtAndDiscardLineKey` (renamed from `TestShell_ScriptExt`) pins each dialect's key.
- The `tmux`-tagged reed tests pass against a real tmux.
