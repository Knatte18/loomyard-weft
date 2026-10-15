# reed: the viewer follows the run until the operator moves it

A reed session's viewer now follows the run by default.
At run start it goes to the overview, and it moves to each newest agent window as the run spawns one, from the shell too.
Before this, a viewer attached before any overview or agent existed rested on `shell` for the whole run, because attach selected nothing and a new agent window only took the viewer from a few window positions.
A second attached client, such as VS Code's attach chain, also suppressed attach's select.

## What changes

- Follow state is the tmux session option `@lyx_follow`: `off` means off, anything else (unset included) means on, so sessions booted by an older lyx follow.
- A new agent window takes the viewer exactly when follow is on, wherever the viewer is.
  A `Focus` strand always takes it and a `KeepViewer` strand never does, whatever the follow state.
  The old viewer-position branches in `newAgentWindowSelected` are gone.
- Attach selects the relevant window with follow on, also beside another attached client; with follow off it keeps the old rule.
- Alt+Up, Alt+Down and a status-bar click on a window, shell or strand button write `@lyx_follow off` in the same tmux command list as their window change.
  Alt+z, Alt+Left/Right and session-button clicks leave it alone.
- The session line of the status bar gets one `follow` button, lit while follow is on.
  A click from on turns follow off without changing window; a click from off turns it on and jumps to the relevant window.
  The off-click is a deliberate addition to "clicking turns follow back on", so the operator can stop following without moving.
- New verb `lyx reed follow`, run by the button through `run-shell -b`: it sets follow on and selects the newest live agent window, else the overview, else nothing.
  Like `lyx reed switch` it is told its geometry and runs no stencil-seed pass.
- `Engine.FollowRun` does the same for loom: every `lyx loom start` entry calls it once after `ensureStatusStrand`, so a reused pair's next run and a resume through `lyx loom start` re-arm follow.
  `lyx loom resume` does not.

## Design decisions to know

- `@lyx_follow` is the one decision-bearing session option in reed, because the manual bindings run inside tmux with no `lyx` process and need a store they can write atomically with `select-window`.
  `@strand`, `@strand_color`, `@lyx_unseen` and the wait marks stay display only, so no PATTERN entry is added.
  The reed doc names the exception.
- A failed read of the option counts as off for that one decision and is logged at Warn; a failed write or select is logged and never fails the operation it rides on.
- Reed's own moves off an ending window (`RemoveStrand`, `ReplaceStrand`, reconcile) neither depend on follow nor change it.
- A window change made outside reed's bindings (tmux prefix keys, a custom `tmux_conf` binding) leaves follow on.
- With follow on, attach moves every attached client of the session, since clients share the current window.
- Every `lyx loom start` entry may move a viewer who turned follow off, including an orch- or batten-driven resume.
- `lyx reed follow` is skipped in the reed command's `PersistentPreRunE` working-directory resolution, like the other binding verbs.
- The button's `run-shell` names `lyx` by absolute path per `PATTERN-pane-binary-resolution`; that pattern's list of bindings gains the follow button.
  If the command cannot be composed, a click from off only sets follow on and logs at Warn.
- Windows and psmux: a psmux without `bind-key` or `run-shell` never sets follow off, so it always follows.
- Out of scope: the configurable button set (note `reed-status-buttons`) and any batten change.
- Docs: `internal/reedengine/doc.go`, `internal/reedcli/doc.go` and `internal/loomcli/doc.go` are updated, and `cmd/lyx/testdata/test-budget.yaml` is edited for the new tests.

## To check by hand

The real-tmux and smoke tests this change edits or adds were compiled under `go vet -tags tmux` and `-tags llm` but not run.
Run them before landing:

- `internal/reedengine` tests with `-tags tmux`: `follow_integration_test.go`, `bindings_integration_test.go`, `ownwindow_integration_test.go`, `windoworder_integration_test.go`.
- `internal/loomcli` smoke tests with `-tags llm`: `smoke_drivingsurface_test.go`, `smoke_driverstrand_test.go`.

Findings left open by review:

- `lyx loom resume` versus the re-arm: confirm that only a resume through `lyx loom start` turns follow on, and that `lyx loom resume` leaves it as it is.
- Failed client listing and a stale `attachArgv` doc in attach (`internal/reedengine/attach.go`, `AttachArgv`): check the failure path and that the doc comment matches the new rule.
- Real-tmux test for attach: later attaches are not waited for, so check the test is not racy.
- The fallback-row tests for the follow button (`bindings_test.go`, `bar_integration_test.go`) may exercise a state the bar cannot reach.
- A click on a button of a session whose name needs quoting: check that the session-line click reaches `lyx reed follow` intact, and that the test has a status line to click on.
- An automatic switch can race a manual one by one switch: a binding's off write landing after reed's read of on lets one more automatic move through.
