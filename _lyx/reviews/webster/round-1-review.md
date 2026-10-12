---
verdict: APPROVED
---

No finding.
The range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/runs-overview/status.json` gives `product.parent` as `main`, and `_lyx/loom/rework/` does not exist, so no live rework round narrows it.
The twelve commits are one per plan card.

Checks that ran:

- `lyx gate test` over the touched packages (statuspane, reedengine and render, termwindow, battencli, battenshed, orchcli, loomcli, loomrecipe, shedrun, seatengine, cmd/lyx) is green, `cmd/lyx` carrying the test-budget and daemon-wakeups scans.
- A per-package count of top-level `Test` functions, excluding `TestMain`, equals its row in `cmd/lyx/testdata/test-budget.yaml` for every package the diff touches, and that file is edited in the commits that raise a row.
- The integration and tmux tiers were not run here; their files were read.

## Rejected

- Top-strand placement (card 2): `planTopPaneTarget` in `internal/reedengine/selvagepane.go` splits below the physically lowest live top pane by `Top`, else the physically top-most pane with `-f -b`, and never reads the layout string, as the physical-top-placement decision requires.
  `bindingsFixture` and `TestZoomBracket_*` exercise the runs-above-orch shape (a top content strand, then a below-parent one) against real tmux.
  A non-top split can still pick a top pane as its target, but only when it is the tallest live non-Selvage pane, and that shape is exercised there.
- Title row (cards 1 and 2): `readBorderTitleRowLocked` is called at exactly two sites, `applyLayoutLockedOpts` and `AttachArgv`, and the one answer feeds `planLayout`, `fixedHeightPins` and `pinsForContent`, as the title-row-once decision requires.
- Content-rows report (cards 3 and 5): `ReportFromEnv` reports with `agentname.StrandNameEnv` and `TMUX_PANE`, and returns nil when either is empty.
  `reedengine/panebin.go` exports the name to strand panes.
  `SetContentRows` refuses `rows < 1` before taking a lock, then an unknown name, a non-content strand and a pane mismatch, and a repeated count touches neither state nor tmux.
  It runs under `withOpLockKeepingZoom`, and `zoom_integration_test.go` asserts that a report keeps the zoom.
- Selvage cycle (card 4): the console marker `@lyx_console` is display-only and is set only on the recorded Selvage pane.
  The step and zoom bindings stay gated on `@lyx_strands`, so other windows' cycling is unchanged.
- Minute resolution (cards 5, 7 and 9): `FormatMinutes` is the only time format the overview and checklist use.
  `RenderStatusLine` carries activity text only, no clock.
  `Pane.Run` doubles its wait up to 8s while the text is unchanged and resets it to 1s on a redraw.
  `statuspane/pane.go` is listed in `unboundedWaitFiles` for the daemon-wakeups scan.
- Notice log (card 8): `appendNoticeLog` runs at the first delivery attempt per composed notice, in `deliver` when the episode delivery is created and in `rowNotifier.send`, so retries do not re-log.
  Tests pin one line for three attempts, two for a restarted producer, an unwritable path only warning, and whole lines under concurrent appends.
- Run window (card 12): `attachLineWaiting` exits on status 0, otherwise waits up to the bound for the worktree directory to go away and falls back to `exec bash`.
  Unit and integration tests pin the exact line.
- Test economy: no new test duplicates an existing one that I found.
  The new top-level tests each cover a distinct public surface (render, `SetContentRows`, pane loop, status cache, overview, checklist, attach line).
