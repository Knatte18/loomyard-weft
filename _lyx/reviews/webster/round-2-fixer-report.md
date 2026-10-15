## Changes

- F1, fixed as proposed (commit `2df730c36`):
  - `internal/reedengine/follow.go`: the `set-option` calls in `FollowRun` and `followSessionVia` now target `exactSessionWindowTarget` (`=name:`); `list-panes -s` stays on `exactSessionTarget`.
  - `internal/reedengine/follow_test.go`, `internal/reedengine/bindings_integration_test.go` and `internal/loomcli/smoke_drivingsurface_test.go`: the pinned argv and the `@lyx_follow` helpers use `=name:`.
  - `internal/reedengine/faketmux_test.go`: the fake answered the session-wide pane listing from its own script even when forwarding to a real server, so `TestFollow_RealTmux/AFailedFollowWriteStillSelects` selected nothing once the write path was right. It now synthesizes that listing only when nothing is forwarded.
- F2, fixed with my own change (commit after `2df730c36`):
  - Proposed change not made: calling `selectRelevantWindowLocked` after the relaunch loop in `Resume`.
    `doc.go` states that `Resume` never selects the relevant window and lists three automatic switch points; the decision record says resume does not turn follow on by itself.
    The window `@2` is the relaunched `dying` own-window strand: with follow on after the restart, each relaunched own window takes the viewer in turn, so the viewer ends on the last one opened, which is the documented behaviour.
  - `internal/reedengine/ownwindow_integration_test.go`: the assertion now expects the highest-index window of an own-window strand, dead or alive.

Verified green with `lyx gate test --tags tmux` on `./internal/reedengine`, `./internal/reedcli` and `./internal/loomcli`.

## Gate finding

The verify run failed `internal/shedverbs` `TestUntilStop_EnvelopeCurrency/a_halt_envelope_is_stale_once_the_file_reads_running_again` (the spawned loop reported as exited before it recorded itself).
This round changes no file under `internal/shedverbs` or anything it imports.
The package passes alone, and 300 runs of the test at `-cpu 1,8` plus 30 more all pass, so I could not reproduce it and made no change; it looks like a timing race in the test's fake loop under load.

## Disputed

None.

## Deferred

None.
