# reed-multi-window: own-window placement end to end

All three batches recorded `done`, with no recovery needed.

## Batch 01-render-own-window-display (cards 1–3, head `7e6f73f16`)

Render accepts the `own-window` display with its window group fields.
`reed.yaml` gained `window_share_max` and `window_split`, validated at up, resume, add and replace.
Reed reads own-window panes through a session-wide listing and reconcile ends own windows no strand holds (`internal/reedengine/ownwindow.go`).
Informational scope warnings only: `reconcile.go` and `render/types.go`.

## Batch 04-reed-own-window-launch (cards 4–7, head `bd21acd2f`)

Reed opens own-window strands in detached windows through the launch chokepoint, and `lyx reed add --anchor own-window` works.
Groups share one window up to `window_share_max`.
Removal moves a viewer to window 0 and relays out a shared window.
The status bar shows own windows in their segment color, underscored until first selected, cleared by a `session-window-changed` hook.
Deviations from declared file-ops: `panebin_enforcement_test.go` (allowlist entry for `spawn.go`), `faketmux_test.go`, `windowsize.go`, `lifecycle.go`, `replace.go`, `state.go`, `ownwindow.go`, `ownwindow_test.go`, `reedcli/remove.go`.
Only compilation of the new tmux-tier tests was checked; they did not run.

## Batch 08-shuttle-present-strand-removal (cards 8–13, head `1837ca953`)

Shuttle removes dead own-window strands and keeps a died run's last capture.
Shuttle runs open in their own window by default, and the orch names window 0.
Seat tables declare one window group sized by their seat count.
The pane shell seam gained `OrElse` and `Exit`.
The provider seam opens a named session with no prompt, and `lyx shuttle open` opens one in its own window with no turn.
Deviations from declared file-ops: `shuttleengine` (`attach.go`, `stepstop.go`, `engine.go`, `wait.go`, `open.go`, `open_test.go`, `fakes_test.go`, `spec_test.go`), `shell/posix.go`, `shell/pwsh.go`, `claudeengine/command.go` and its test, `seatengine/prompt_test.go`, `shuttlecli/run.go`, `shuttlecli/open.go`.
The fork reports appending to three `internal/shell` files with a Bash heredoc, which the edit rule disallows; the audit raised no finding.

The plan-level verify is left to the gate.
