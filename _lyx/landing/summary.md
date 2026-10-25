# The overview window becomes the beam window

The orch's overview window is now the beam's own window, named `beam: <slug>`, or `beam: prime` in the prime.
Its name used to say neither that it belongs to a beam nor what it covers, and the runs showed in two places: the run list and the dobby section.
The frame `lyx batten overview` draws is now a dobby section, a blank line, then an orch section; each run shows once, in the dobby section.

## What changes

- The separate list of the prime's runs in flight is removed.
  Each dobby run line reads slug, state, stage and time in stage as the run list drew them, then the dobby's pass cell.
  A covering pause reads `paused <minutes>`, a run in `Run-Shed` shows the child's numbered stage, and an unreadable status reads `status unreadable`.
  Run age and reed session are dropped to keep the line short.
- `lyx beam status` prints the same dobby lines the pane draws, from one function that renders the cells for both.
- The orch section shows the orch of the window's own worktree and of every beam below it, found by walking `hubgeom.ParentOf`; the prime's window shows every orch of the hub.
  A worktree with no dobby, such as a task pair, draws the orch section alone.
- Docs updated: `internal/orchengine/doc.go`, `internal/reedengine/doc.go`, `internal/beamcli/doc.go`, the module comment in `internal/battencli/cli.go`, and `docs/overview.md`.

## Design decisions

- `render.Display` gains an optional `Title`, read only for an overview strand.
  Reed names the overview window `Title`, and `overview` when it is empty, so the loom status strand of a task pair keeps its `overview` window.
  Reed cannot tell a prime from a beam, and the display is persisted with the strand, so a relaunch keeps the name.
- The strand keeps its `runs` role and the verb stays `lyx batten overview`.
  `lyx orch start` finds and replaces an existing session's strand by that role, so the new window name reaches running sessions without a migration.
- The ancestor walk in `atOrBelow` is guarded against a cyclic parent chain.

## To check by hand

- A session started by an older build keeps its `overview` window until `lyx orch start` replaces the strand.
- The window name contains a colon and a space.
  Reed targets the window by id after creation, and the tests cover that; confirm in a live tmux session that the name displays and the window orders correctly.
