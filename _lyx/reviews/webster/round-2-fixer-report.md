## Changes

All eight findings fixed as proposed unless marked; each commit is individual except F4 and F5, which share one file and one commit.

- F1: `smoke_selvage_keepalive_test.go` — the bottom-most subtest became `StaysSoleInShellWindowAcrossAddsAndRemoves`, asserting via `list-panes -t <selvage pane>` (window-scoped) that Selvage is the only pane of the shell window after each add and remove; header comments updated.
- F2: `attachgeometry_integration_test.go` — the degraded-path test now requires the bare prefix plus the 4-element `; select-window -t <window>` tail.
- F3: `bar_integration_test.go` — the fixture reads the extra window's index from tmux, and both `range=window|<n>` expectations use it.
- F4: `bindings_integration_test.go` — the Alt step sequence follows the real window order (overview, alpha, beta, extra, agent, shell; hidden windows skipped), read from tmux rather than the review's guessed order.
- F5: `bindings_integration_test.go` — fixed with my own change: the shell-click check now selects alpha's window, clicks shell and waits for the shell window to become current. The proposed zoom of the Selvage pane is not possible, because tmux cannot zoom a sole pane. The orphaned `selectStrandsWindow` and `zoomed` helpers were removed.
- F6: `activity_integration_test.go` — the strand's window and pane are selected after `waitForClientAttached`.
- F7: `paste_integration_test.go` — the sink touches a ready file after `stty`, and the test waits on it before pasting.
- F8: `ownwindow_integration_test.go` — the `leftover` helper gives its strand `ResumeCmd: "sleep 300"`. With a live relaunch command the subtest passes, so no product defect in relaunching a moved dead pane.

Verified with `lyx gate test --tags tmux ./internal/reedengine ./internal/reedcli`: both packages ok.
The integration tier (`--tags integration ./...`) was not run.

## Disputed

None.

## Deferred

None.
