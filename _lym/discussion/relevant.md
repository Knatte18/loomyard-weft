# Relevant files

- `internal/termwindow/termwindow.go` `Resolve`, `Launcher.OpenAttach` — builds the Konsole command and starts the window; gains `--separate` and the process id
- `internal/reedcli/window.go` `openTerminalWindow`, `placeTerminalWindow` — the `lym reed window` path that opens and then runs the window hook
- `internal/desktophook/window.go` `AfterWindow` — the existing after-window command, kept window-manager-free
- `internal/desktophook/template.yaml` — the desktop module's template, where the new machine keys go
- `internal/machinecli/start.go` `openWindows` — opens one window per started worktree on `lym machine start`
- `internal/machinecli/cli.go` — wires the terminal opener for `lym machine start`
- `internal/hubgeom/parentof.go` `ParentOf` — names a worktree's parent, the anchor of its window
- `internal/hubgeom/tasksetting.go` `TaskSettingChain` — the chain the per-task beam setting resolves over
- `internal/dobbyengine/tasksetting.go` `LookupRunWatch`, `LookupWithSource` — the one per-task beam setting lookup the new setting joins
- `internal/dobbyengine/template.yaml` — the beam template where the per-task setting goes
- `internal/battencli/arm.go` `reopenWindow` — reruns the Open-Window command on resume under a 30 second bound
- `internal/proc/proc_linux.go` `Detach` — starts Konsole in a new session, so the started process id is Konsole's
- `pattern/PATTERN-config-layers.md` — machine layer and per-task beam setting rules
- `pattern/PATTERN-test-stand-ins.md` — rules for the KWin and kscreen stand-ins
- `pattern/PATTERN-lym-files.md` — where the places cache lives
