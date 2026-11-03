# Show the board handle in run window titles

The operator names runs by board handle (`T4`, `P1`), but no window showed it.
A task run's desktop terminal window was titled with the slug, and its tmux overview windows carried the recipe or `beam: <slug>` alone.
Each of these windows now opens with the run's handle, read from the board when the window opens:

- Batten's Open-Window row titles the desktop terminal window `<handle>: <slug>`, such as `T4: webster-subpath-anchor`.
- The loom status strand's overview window is titled `<handle>: <recipe>`, such as `T4: darn`.
- A beam's runs strand overview window is titled `<handle>: beam: <slug>`, such as `P1: beam: <slug>`.
  The prime's `beam: prime` is unchanged.

When the board cannot be read, or has no handle for the slug, each title falls back to today's name and the run goes on.
A window opened by hand with `lym reed window` and no `--title` keeps the directory's base name.

## Design decisions

- `boardengine.TitleWithHandle(boardDir, slug, name)` is the one place a title gets its handle.
  It reads the local board file, logs a read failure at Warn, and returns `name` unchanged on any failure, so a title never fails its caller.
- The status window is titled `<handle>: <recipe>`, not `<handle>: overview`.
  The window was already titled by its recipe, which lets a beam session tell a pawl checklist window from a run's own, so the handle is put in front of the recipe.
  This departs from the task body's `T4: overview` example.
- `shedadapters.CommandDeps` gains `Title`, behind a new `{title}` placeholder for `open_window_command`, `seed_ide_command` and `open_ide_command`.
  It is called only when a command holds the placeholder.
  The slug is the fallback, so `{title}` is `<handle>: <slug>` or the slug alone.
- The default `open_window_command` in the batten template now passes `--title {title}` in place of `--title {slug}`.
  A command already set in the board default layer keeps its own `{slug}`, so that window stays titled with the slug until the value is changed.
- `orchCLI.runsWindowTitle` is now a `func() string`, so the beam's handle is read when the runs strand is added or replaced.
- A live strand of the running build is kept as it is, so a handle the board assigns after a window opened shows from the strand's next replacement.
  The package docs of `loomcli` and `orchcli` say so.
- `boardkit.WriteBoard` is a new test helper that writes a board file holding given entries without a Board.

Tests pin the titled and the fallback names of the Open-Window title, the status window and the runs window, and `TitleWithHandle` itself.
The package docs and `docs/overview.md` describe the new titles.

## To check by hand

- `battencli-title-wiring-untested`: the `Title` closure that `commandDeps` in `internal/battencli/wire.go` wires into `shedadapters.CommandDeps` has a unit test (`internal/battencli/commanddeps_test.go`), but no test runs it through a real Open-Window row to `lym reed window --title`.
  Open a task run and confirm the Konsole window reads `<handle>: <slug>`.
- `status-title-departs-unrecorded`: the status window is titled `<handle>: <recipe>`, not the `<handle>: overview` the task body names.
  Confirm that is the intended title.
- The `reed`/`tmux` window names and the Konsole title were not checked in a live session: start a loom run and a beam and read the window names.
