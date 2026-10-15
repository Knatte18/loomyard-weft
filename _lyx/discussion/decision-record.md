# Decision record — reed-launch-fixes

## Goal

Concurrent `lyx shuttle run` launches into one tmux session each start their agent from an intact launch line, and `lyx shuttle run --focus=false` never switches the viewer's window.

## Scope

- In: reed's own-window open, its "does the new agent window take the viewer" rule, and the keys `launchStrandLocked` types into a fresh pane.
- In: `lyx shuttle run --focus` reaching that rule as a strict "never take the viewer" when false.
- In: the shell dialect gaining the key that discards a pane shell's unsubmitted input line.
- Out: a flag that opens a run's window outside the caller's session, and the `--anchor hidden` startup failure; the board entry puts them in scope only if the fix needs them, and it does not.
- Out: `lyx reed add --focus`, whose default false keeps meaning "follow the run".

## Decisions

- **`render.Display` gains `KeepViewer`.**
  Rationale: `Focus` false already means "follow the run" for every internal spawner, which leaves a viewer on the relevant window (the orch's, in the prime) moved to each new agent window; a separate flag lets a caller ask for "never move" without changing what the zero value means.
  Bound: it only suppresses the select of an own-window strand's new window; a set `Focus` still wins, and removing a viewed window still moves its viewer.
- **`lyx shuttle run` sets `KeepViewer` to the negation of `--focus`.**
  Rationale: the flag defaults to true, so `--focus=false` is always an explicit ask to stay put.
- **A new agent window that keeps the viewer carries `@lyx_unseen`, as any unselected agent window does.**
  Rationale: the unseen mark is how the operator learns a window opened without being shown.
- **`launchStrandLocked` sends the dialect's discard-line key before the literal launch payload.**
  Rationale: a fresh pane's shell can receive input meant for the viewer's active pane (keystrokes, terminal replies) when the viewer is switched onto it while it boots, and those bytes prefixed the typed launch line (`5. '<script>'`); discarding the pending line first leaves only the launch line.
  Bound: the key only discards the unsubmitted input line of a pane reed just created, before reed typed anything into it.
- **The discard key is `C-u` on POSIX and none on PowerShell.**
  Rationale: `C-u` discards the line in the tty's canonical mode and in readline alike; PowerShell's equivalent, Escape, would merge with the typed `.` into an Alt chord, so pwsh sends nothing extra.
- **The key lives in `internal/shell` as `DiscardLineKey`.**
  Rationale: which key discards the line is a fact of the shell dialect, chosen on the same `shell.ForGOOS()` the launch line uses.

## Constraints

- `PATTERN-shell-mechanics-seam`: `internal/shell` stays stdlib-only.
- `PATTERN-guarded-interrupt`: the discard key is never the interrupt key.
- `PATTERN-tolerant-records`: the new `Display` key is `omitempty`, and older `reed.json` records without it read as false.
- Docs: `internal/reedengine/doc.go`'s focus rules name the new flag in the same commit.

## Auto-mode assumptions

- The stray digits came from input routed to the fresh pane while the viewer hopped onto it; the logs show the launches serialized about 100 ms apart, each of them a `--focus=false` run selected by the follow rule.
- No other caller of `lyx shuttle run --focus=false` relies on it following the run.

## Open risks

- Stray input can still reach a booting pane when a `Focus` or follow-rule window takes the viewer; it is discarded only when it arrives before the `C-u`.
- PowerShell panes get no discard key, so the launch-line part of the fix is POSIX-only.

## Acceptance criteria

- `newAgentWindowSelected` returns false for a `KeepViewer` display on every current window that would otherwise pass the viewer on, and true when `Focus` is also set.
- `lyx shuttle run --focus=false` builds a spec whose display has `KeepViewer` true and `Focus` false; the default builds `Focus` true and `KeepViewer` false.
- A POSIX launch types `C-u`, then the literal source statement, then `Enter`, as three separate `send-keys` calls.
- A PowerShell dialect's `DiscardLineKey` is empty, and its launch types only the payload and `Enter`.

## Notes for the plan writer

- `internal/reedengine/ownwindow.go`: `newAgentWindowSelected`, `newAgentWindowTakesViewerLocked`.
- `internal/reedengine/spawn.go`: `openOwnWindowLocked`, `launchStrandLocked`.
- `internal/shuttlecli/run.go`: the `--focus` flag.
- Tests: `TestNewAgentWindowSelected` in `ownwindow_test.go`, `TestLaunchStrandLocked_LaunchScript` in `spawn_test.go`, `internal/shell/shell_test.go`.
