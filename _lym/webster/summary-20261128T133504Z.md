# KDE Plasma 6 window placement

All three batches of the plan are recorded `done`: `01-kwin-probes`, `05-konsole-separate-launch` and `11-kwinplace-place-flow`.
The head after the last batch is `f525096fd28c477e09b6092ca26ca61d2dbe57f7`.
No digest reported a departure or a rename.
The deviations below are the files the scope guard found changed outside a batch's own targets, and are informational.

## 01-kwin-probes

The batch covers the KWin probes and their recordings, the `beam.window_placement` setting, the desktop placement keys and the cancellable window command.
It stopped several times as `waiting` while the operator ran the probes.
During those stops the parent recorded decisions on the screen setup the probes run on, and a final one on D6's clamp:
a window in a zone that runs under a panel stays tiled, with no release and no frame setting.
That clamp decision applies over the plan's card text.

Deviations: `internal/desktophook/config_test.go`, `internal/desktophook/window.go`.

## 05-konsole-separate-launch

The batch covers the separate Konsole launch and the `internal/kwinplace` scripts, readers, pre-launch checks, script runner, remembered places and planner.

Deviations, all under `internal/kwinplace/`:
`places.go`, `places_test.go`, `planner.go`, `planner_test.go`, `prelaunch.go`, `recordings_test.go`, `report.go`, `runner.go`, `runner_test.go`, `scripts.go`, `snapshot.go`.

## 11-kwinplace-place-flow

The batch covers the placement flow, the detached placement child, and handing the windows of `lym reed window` and `lym machine start` to it.

Deviations:
`internal/kwinplace/child.go`, `internal/kwinplace/flow.go`, `internal/kwinplace/flow_test.go`, `internal/kwinplace/parentwait.go`,
`internal/machinecli/start_test.go`, `internal/reedcli/placewindowchild.go`.
