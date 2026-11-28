# Place lym's Konsole windows on KDE Plasma 6 desktops and tiling zones, remembered per hub

`kde-window-placement-build` · Task · feature, termwindow, desktop, reed, dobby, cwd, fabric, gate, pattern, docs · running · Code-Burler-Review

Decision record: [Sketch](decision-kde-window-placement-build.md)

Filed from the Sketch `kde-window-placement`.

Takes: termwindow's Launcher.OpenAttach and Resolve, desktophook's AfterWindow runner and config, dobbyengine's per-task lookup over hubgeom.TaskSettingChain, hubgeom.ParentOf, state.UpdateJSON, lock.AcquireWriteLock, standalonestate.UserStateDir, selfexec.LymBinary, runproc.Register, lymtemp, and the layout as fabric-core-foundation and whole-repo-guards-off-gates land it; the KWin facts Open risks lists, probed as the run's first step, each with a KWin script run through `qdbus6` and its console.log lines read from the journal: the read-only probes run first, the moving probes only once the operator runs or approves them, and a probe that contradicts a listed fact stops the run for the parent, with the probe's command and output, before any code rests on it
Delivers: D1's pre-launch checks and --separate Konsole launch returning its pid; internal/kwinplace (new) with the script runner, script lock, query cache, kscreen reader, pure planner, places file and placement child; the hidden placement verb that lym reed window and lym machine start hand each window to; desktop.window_placement, desktop.tiling.min_tile_width and min_tile_height, and beam.window_placement; a cancellable window_command runner; the real-desktop contract test in a manual tier
Exports: kwinplace's placement entry and planner, termwindow's pid-returning launch and pre-launch check, dobbyengine.LookupWindowPlacement, the three desktop keys

The part changes these packages:

- internal/kwinplace (about 2600 changed lines)
- internal/termwindow (about 120 changed lines)
- internal/desktophook (about 150 changed lines)
- internal/reedcli (about 200 changed lines)
- internal/machinecli (about 80 changed lines)
- internal/beamsetting (about 30 changed lines)
- internal/dobbyengine (about 100 changed lines)
- internal/fabricengine (about 20 changed lines)
- internal/lymdirs (about 10 changed lines)
- internal/gateslot (about 20 changed lines)
- cmd/lym (about 30 changed lines)
- pattern (about 10 changed lines)
- docs (about 20 changed lines)
