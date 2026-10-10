# Card 10 — resize-signal-entry-at-boot

**Create:**
- `plan:internal/reedengine#Engine.pinBootOptionsLocked` -> `func (e *Engine) pinBootOptionsLocked(mouse, shellPath string) error`

**Edit:**
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine#Engine.installResizePinsLocked`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#Engine.pinGeometryOptionsLocked`
- `internal/reedengine#Engine.resizeSignalHookCommand`

**Intent:**
Decision D4: on POSIX with `watchdog: on`, a fresh boot installs the window-resized hook array as a zero-pin rebuild, the clear and the signal entry alone, so a session with fewer than two panes or no placed strand promotes its watcher out of poll mode without waiting for a layout apply.

`pinBootOptionsLocked` is the tail of `ensureServerAndSessionLocked` after the boot loop and the discover-signal touch, lifted into one method so a fake tmux can reach it: the `remain-on-exit` and `mouse` pins with their existing fatal errors, then `pinGeometryOptionsLocked(target, shellPath)`, then, when `resizeSignalHookCommand()` is non-empty, which is exactly POSIX with the watchdog on, `installResizePinsLocked(target, nil)`.
`ensureServerAndSessionLocked` calls it in place of the lifted block and returns its error.
The call sits in the boot path and never inside `pinGeometryOptionsLocked`, which the attach pre-flight also runs: there a degrading attach returns before its install, and a zero-pin rebuild would wipe the session's resize pins.
The attach pre-flight keeps its order, and a later layout apply or successful attach rebuilds the array with its pins as today.
POSIX with `watchdog: off` keeps the unset `pinGeometryOptionsLocked` already issues, and Windows issues no hook call.

`installResizePinsLocked`'s doc comment, which says the signal entry reaches exactly the sessions an apply reaches and a session that never had an array keeps its watcher in poll mode until the first real apply, is reworded: the boot installs the signal entry for every session, and an apply or attach rebuilds the array with its pins.
`doc.go`'s resize-pin paragraph says the same in its "the paths that install nothing" and "a session that has never reached an install" sentences.

Test: a new `TestPinBootOptionsLocked_InstallsTheSignalEntry` in `lifecycle_test.go` with three steps on a fake tmux: `WatchdogOn` asserts the `set-hook` calls are the `-u -w` clear followed by the entry whose body is `resizeHookCommand(shell.ForGOOS(), e.resizeSignalPath())` and nothing else, after the geometry pins; `WatchdogOff` asserts the one `set-hook -u` unset and no `-w` entry; both are skipped on Windows, and a `Windows` step is not written since `runtime.GOOS` cannot be faked, so the Windows branch is covered by `resizeSignalHookCommand`'s existing test answering empty there.
The attach side, that a degrading pre-flight issues no `set-hook`, is already pinned by `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` and stays.
The `internal/reedengine` row in `test-budget.yaml` rises by one, to 247.

**ImpactSummary:** The boot path's post-spawn pins move into one method and gain one hook install; the attach pre-flight, the apply path and the watch loop are untouched, and a strand-less session's watcher now promotes to signal mode at boot instead of polling for its whole life.
