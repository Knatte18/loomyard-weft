# Run windows show the board handle in their titles

The operator names runs by board handle (`T4`, `P1`), but no run window showed one.
Each run window now opens its name with the handle, read from the board when the window opens:

- **Task run terminal window.** The Open-Window row's default command is now `{lym} reed window {task_dir} --title {title}`.
  `{title}` is a new placeholder of the `Command` producer (`internal/shedadapters`), filled lazily through `CommandDeps.Title`.
  batten fills it with `<handle>: <slug>`, for example `T4: webster-subpath-anchor`.
  The other Command rows may use it too.
- **Loom status window.** `lym loom start` titles the `loom-status` overview window `<handle>: <recipe>`, for example `T4: tapestry`.
- **Beam runs window.** A beam's runs strand window is `P1: beam: <slug>`.
  The prime's stays `beam: prime`, since the prime has no board entry.

`boardengine.TitleWithHandle(boardDir, slug, name)` builds all three titles.
It reads the local board file and returns `name` alone when the board holds no entry for the slug, the entry has no handle, or the board cannot be read.
An unreadable board is logged at Warn, so a missing handle never fails a row or a bring-up.
`lym reed window` with no `--title` is unchanged and still uses the directory's base name.

## Design decisions

- **The status window opens the recipe, not `overview`.** The board entry asked for `<handle>: overview` in place of the bare `overview` default.
  Since the entry was written, the landed pawl-checklist change (`7c23591e`) titles that window with the run's recipe.
  So the handle opens the recipe name, giving `T4: tapestry`, and the window keeps telling a pawl checklist apart from a task run's.
  The bare `overview` default in `internal/reedengine/windoworder.go` remains for an overview strand with no title.
- **The orch reads the runs title lazily.** `orchCLI.runsWindowTitle` is now a function that `ensureRunsStrand` calls, so the board is read only when the strand is added or replaced and not on every `lym orch` verb.
- **A hub override keeps its own command.** A hub whose board default layer sets its own `open_window_command` keeps that command.
  To get the handle in the title, it needs `{title}` in place of `{slug}`.
- **The status-bar `term` button is unchanged.** It still runs `lym reed window {worktree} --title {slug}` from reed's button template, because the entry covers only the window Open-Window opens.

## Tests

- `TestTitleWithHandle` (boardengine) covers the titled name and each fallback: no entry for the slug, an entry with no handle, an unreadable board and no board.
- `TestStatusWindowTitle` (loomcli) covers `T4: tapestry` and the bare recipe for an entry of another slug and for no board.
  `TestStatusStrandAddSpec` covers a handle-titled window name.
- `TestRunsWindowTitle_WiredFromTheWorktreeTheVerbRunsIn` (orchcli, tier 1) covers the wired title before and after the board names the beam: `beam: b`, then `P1: beam: b`, with the prime staying `beam: prime`.
- `TestCommandProducer` gains a row showing that `{title}` reaches the shell as one quoted word.
  The batten template test expects the new default command.
- `boardkit.WriteBoard` is a new shared test helper that writes a board file holding given entries.
