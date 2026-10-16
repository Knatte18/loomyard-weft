# Every strand reed places gets a window of its own

All six batches recorded `done`.
Each batch's `record-batch` returned `ok: true` with no warnings.
The scope guard listed changed files outside each batch's own targets; they are informational and are named below.

- **01 retire-resize-watch-loop** (head `b6c21641d`): the per-worktree resize watch loop and its re-apply are gone, and the daemon keeps name repair.
  Deviations: `internal/reedcli/watchdog.go`, `internal/reedcli/watchdogreap_integration_test.go`, `internal/reedengine/namerepair.go`.
- **02 retire-shell-window-stack** (head `9f44bda69`): render's layout, reed's shell-window apply, the pins, the hook array and the attach chain are removed; boot and attach unpin `window-resized`.
  Deviations: 13 files, all under `internal/reedengine` (including `render/types.go`), mostly tests plus `apply.go`, `attach.go`, `lifecycle.go`, `selvagepane.go`, `spawn.go` and `watchdog.go`.
  The fork reported dropping the `sizeKnown` lock choice in `AttachArgv`.
- **03 retire-zoom-and-pane-buttons** (head `520510a9c`, cards 3 and 4): the zoom bracket, the strand pane buttons and click are gone, Alt+z is unbound, and `collapsed_rows`, `min_full_rows` and `selvage` are retired through `RetiredKeys`.
  Deviations: `internal/reedengine/lifecycle.go`, `internal/reedengine/windowsize_test.go`.
- **05 retire-pane-era-anchors** (head `3ff7271c8`): `below-parent` and `top` are remapped to `own-window` on read and refused on write, and `reed add` defaults to `own-window`.
  Deviations: `internal/reedcli/smoke_test.go`, `internal/reedcli/watchdogreap_integration_test.go`, `internal/reedengine/attach_test.go`, `internal/reedengine/retiredkeys_test.go`, `internal/reedengine/windowtarget.go`.
  The fork recorded a friction note: a rewrite of a retired `reed.json` does not keep the old `size` and `contentRows` keys, because `ReedState` has its own decoder.
- **06 shuttle-run-anchor-vocabulary** (head `10ba8dd55`, cards 6 and 7): `lyx shuttle run --anchor` accepts only `own-window|hidden`, and the reconcile head breaks a strand pane left in the shell window out into its own window.
  Deviations: `internal/reedengine/faketmux_test.go`.
  The fork reported appending one function with a shell heredoc instead of Edit or Write, against the editing rule; the gates passed on it.
- **08 sweep-docs-and-sandbox** (head `66dabb434`): the package docs, overview and sandbox suites are rewritten, and the #673 workaround is removed from `CLAUDE.md`.
  Deviations: `internal/shuttlecli/cli_test.go` (two subtests renamed so the plan's first verify grep passes).
  The fork noted a stale comment at `internal/reedengine/server.go:28` that still mentions the resize watch loop.

The forks report the `tmux` tier compiled with `go vet` but not run, so the new real-tmux subtests have not executed.
I did not run the plan-level verify; the gate does that.
