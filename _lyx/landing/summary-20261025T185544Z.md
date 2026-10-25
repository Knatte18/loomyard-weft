# The overview window becomes the beam window

The orch's `runs` window is now the beam's own window, named `beam: prime` in the prime and `beam: <slug>` in a beam; before, it was `overview`, which named neither the beam nor what the window covers.
Its frame is two sections: the dobby, then the orch.
Each run shows once, in the dobby section; the separate list of runs in flight is gone, so runs no longer appear in two places.

## What changes

- The dobby section draws one line per run: slug, state, stage and time in stage, then the dobby's pass cell.
  A covering pause reads `paused <minutes>`, a run in `Run-Shed` shows the child's numbered stage, and an unreadable status reads `status unreadable`.
  Run age and reed session are dropped to keep the line short.
- The orch section draws one line per orch: the window's own worktree and every beam whose parent chain reaches it.
  The prime's window shows every orch of the hub.
  Parents are read only through `hubgeom.ParentOf`, and a cyclic or broken chain leaves that orch out.
- A worktree with no dobby, such as a task pair, draws the orch section alone.
- `lyx beam status` prints the same dobby lines as the pane, from one shared cell-rendering function.
- Docs follow: `internal/orchengine/doc.go`, `internal/reedengine/doc.go`, `internal/beamcli/doc.go`, the `internal/battencli/cli.go` module comment and `docs/overview.md`.

## Design decisions

- `render.Display` gains an optional `Title`, read only for an overview strand.
  Reed names the window `Title`, and `overview` when it is empty.
  Reed cannot tell a prime from a beam, and a task pair's loom status strand has no beam to name, so it keeps `overview`.
  The display is persisted with the strand, so a relaunch keeps the name.
- The strand keeps its `runs` role and the verb stays `lyx batten overview`.
  `lyx orch start` finds and replaces a session's strand by role, so the new name reaches a running session on its next start after a build change, with no migration.
- The pane's text still changes at most once a minute when idle, so its poll keeps backing off.

## Check by hand

- A session started by an older build keeps its `overview` window until `lyx orch start` replaces the strand.
- The window name `beam: <slug>` holds a colon and a space; the reed integration test covers targeting the window by id after creation, but confirm in a live tmux session that the name shows and no lookup by name misfires.
- Cyclic parent chain in a beam window (`atOrBelow` in `internal/battencli/overview.go`): the guard and its `ACyclicParentChainIsLeftOut` test were added; confirm a malformed origin record does not stall the pane.
- Orch section in `docs/overview.md` (the orch module row): confirm it reads correctly for the renamed window.
- `lyx beam status` run cells in `internal/beamcli/cli.go`: a test now plants one run; confirm the live output matches the pane.
- Orch line order in `internal/battencli/overview.go`: lines follow the hub's orch order, so a nested beam can precede the window's own orch; the comments no longer claim otherwise.
