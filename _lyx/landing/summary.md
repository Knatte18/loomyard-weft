# Overview window becomes beam: <slug>, with a dobby and an orch section

The window the `runs` strand opens in the prime and in each beam was named `overview` and listed the prime's runs in flight above the worktree's dobby, which lists the runs it drives, so each run showed twice.
It is now the beam's own window, named `beam: prime` in the prime and `beam: <slug>` in a beam, with two sections:

- **Dobby**: alive or not and its `max_runs`, each run it drives once, and each released task it has not started with what it waits on.
  A run line reads slug, state, stage, time in stage, then the pass cell.
  The run keeps the detail the old run list gave it: `paused <minutes>` under a live pause, and the child's numbered stage (`6.6 Plan-Write`) while batten is in Run-Shed.
- **Orch**: one context line per orch, against the soft threshold and the hard cap, with phase and cycle count.
  A beam's window shows its own orch and the orch of every beam below it.
  The prime's window shows every orch of the hub.

`lyx beam status` prints the same dobby lines the window draws, run cells included.

## Design decisions

- `render.Display` gains an optional `Title`, read only for an overview strand.
  Reed names the overview window by it, and `overview` when it is empty, so a task pair's loom status window keeps its name.
  Reed cannot tell a prime from a beam itself, so the caller names the window.
  The display is persisted with the strand, so a relaunch keeps the name.
- `lyx orch start` sets the title from the beam check its pre-run already makes (`runsWindowTitleOf`).
  The strand keeps its `runs` role and the verb stays `lyx batten overview`.
  A session started by an older build gets the new name when the next `lyx orch start` replaces the strand for the new build, with no migration.
- The dobby section comes first and the orch section second, the order the board entry names them.
  A blank line goes between them only when both have lines.
- The run cells live in `battencli.DobbyRunCells`, which holds the status and seed caches the pane's old run poll had.
  `DobbyLines` takes the cells by slug, so the pane and `lyx beam status` render the same thing.
  `dobbyengine.RunView` is unchanged, so the dobby's scheduler reads no child seeds every tick.
- Run age and reed session are dropped from the run line to keep it short.
  Runs are in the dobby view's slug order instead of start order.
- A run's status that fails to read twice draws `status unreadable` in place of its cells and takes no part in the column widths.
- Which orchs a beam's window shows is decided by `orchsAtOrBelow`, which follows each beam's parent chain through `hubgeom.ParentOf` (cached per slug by `DobbyParents`).
  A beam whose chain does not read is left out of a beam's window and logged at Debug.
  When the orch list does not read, the window falls back to its own orch alone; this used to be the prime's orch.
- The cancel integration test's check that a run is gone now reads the prime's dobby view rather than the pane's frame.

## To check by hand

- On a live hub after `update-plugins.sh` and `lyx orch start`, check the first window reads `beam: prime` in the prime and `beam: <slug>` in a beam, with the dobby section above the orch lines.
- In a beam with a nested beam, check its window lists its own orch and the nested beam's, and leaves out the prime's and sibling beams'.
- `internal/battencli/overview.go`, `DobbyRunCells.Of`: it reads the pause record once per frame and each run's status through cached polls; confirm the pane still backs off while idle.
- `internal/reedengine/spawn.go`, `openOwnWindowLocked`: the window name now contains `: ` and a space; reed targets the window by id after creation, and the tier2 window-order test covers a titled replacement.
- `UpdateStrand` accepts a Title change on an overview strand without renaming the window; nothing calls it that way today.
