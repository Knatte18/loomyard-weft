# reed: the viewer follows the run, with a follow button

A reed session's viewer now follows the run by default.
At run start it goes to the overview, and it moves to each newest agent window as the run spawns agents, from the shell too.
Before this, a viewer attached before the overview existed rested on `shell` for the whole run.
Batten's Open-Window attaches before any window exists, and a second attached client (VS Code's attach chain) suppressed attach's `select-window`.

## What changes

- **Follow state.**
  The tmux session option `@lyx_follow` holds it: `off` means off, anything else, unset included, means on.
  Sessions booted by an older lyx therefore follow.
- **New agent windows.**
  `newAgentWindowSelected` takes the viewer exactly when follow is on.
  A `Focus` strand always takes it, a `KeepViewer`-only strand never does, and the viewer-position heuristics are gone.
- **Attach.**
  With follow on, attach selects the relevant window also when another client is attached; with follow off it keeps the old rule.
  With no relevant window yet it selects nothing, and the run start or the next agent window moves the viewer.
- **Run start.**
  A new `Engine.FollowRun` turns follow on and selects the relevant window.
  Every `lyx loom start` entry calls it after `ensureStatusStrand`, so a reused pair's next run re-arms follow.
  `lyx loom resume` leaves the follow state alone.
- **Manual switches turn follow off.**
  Alt+Up, Alt+Down and a status-bar click on a window, shell or strand button set `@lyx_follow` off in the same tmux command list as the window change.
  Alt+z, Alt+Left/Right and session-button clicks leave it as it is.
- **Follow button and verb.**
  The session line carries one `follow` button, lit while follow is on.
  A click from on turns follow off and changes no window; a click from off runs the new `lyx reed follow`, which turns follow on and selects the newest live agent window, else the overview, else nothing.

## Design decisions

- `@lyx_follow` is the one decision-bearing session option in reed; `@strand`, `@strand_color`, `@lyx_unseen` and the wait marks stay display only.
  The bindings run inside tmux with no `lyx` process, so a tmux option is the one store they can write atomically with their `select-window`.
  A failed read counts as off for that decision and is logged at Warn; a failed write or select is logged and never fails the operation it rides on.
- Reed's own moves off an ending window (`RemoveStrand`, `ReplaceStrand`, reconcile) neither depend on follow nor change it.
- The click-while-on toggle is an addition to the board's "clicking turns follow back on", so the operator can stop following without changing window.
- `lyx reed follow` is told its socket, session and tmux path like `lyx reed switch`, and is added to the reed command's `PersistentPreRunE` skip list so it does not resolve the working directory.
  The binding names `lyx` through `panebin.go`; `PATTERN-pane-binary-resolution` lists the new binding.
- Limits, stated in the reed `doc.go`:
  a window change made outside reed's bindings (tmux prefix keys, a user's own `tmux_conf` bindings) leaves follow on;
  every `lyx loom start` entry, a resume of a parked run included, re-arms follow and may move a viewer who turned it off;
  with follow on, an attach moves every attached client of the session.
- Out of scope: the configurable button set (note `reed-status-buttons`), which later takes this button into its set, and any change to batten.

## To check by hand

- The real-tmux (`tmux` tag) and smoke (`llm` tag) tests this change touches or adds were only compiled under `go vet`, not run:
  `internal/reedengine/follow_integration_test.go`, `windoworder_integration_test.go`, `internal/loomcli/smoke_drivingsurface_test.go` and `smoke_driverstrand_test.go`.
- A failed-client-listing path in attach: `internal/reedengine/attach.go` may leave a stale doc on the `AttachArgv` client-listing behavior.
- The real-tmux test of later attaches in `internal/reedengine/follow_integration_test.go` does not wait for those attaches to take effect, so it may be racy.
- The live-fallback row tests for the follow button in `internal/reedengine/bindings_test.go` may exercise a state the code cannot reach.
- A session whose name needs quoting: the follow button's click path has no status-line test for such a name in `internal/reedengine/bindings_test.go`.
- Race, accepted: a binding's off write landing after reed's read of on lets one more automatic move through.
