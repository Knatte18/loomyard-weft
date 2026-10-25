# An anchor per beam

`umbrella-anchor` · Task · enhancement, fabric, config, workspaces

Depends on: [`adopt-runs-tests`](design-adopt-runs-tests.md)

## Seen (2026-10-18)

The anchor (`fabric clone --subpath`) is set once per hub and decides where `_lyx`, `.lyx/` and config sit in every worktree of the hub.
Beams exist now, and all of them still use the hub's one anchor.

## Why an anchor per worktree (operator, 2026-10-24)

The anchor decides where the weft junctions are made and placed inside the matching warp, and where the hub's and each beam's VS Code workspace file roots a worktree; both let work happen in a deep subdirectory without walking far down to it.
All beams share one hub and one board, so a separate hub per subdirectory (`fabric clone --subpath`, a whole new weft repo and board) is no answer.

## Design (operator, 2026-10-24): `.lyx-anchor` at each weft branch's root

- Every weft branch may carry a `.lyx-anchor` at its own root, so the anchor is set per beam and in effect per worktree.
- The weft worktree sits at `<hub>/<slug>-weft`, so lyx finds the marker without knowing the anchor; no anchor index on Bolt is needed.
- Inheritance comes from git: a task's weft branch starts from its beam's weft branch and carries the beam's marker; a worktree with another anchor commits its own.
- No marker stays on Bolt: `fabric clone --subpath` writes the first `.lyx-anchor` at the root of `main-weft`, and every weft branch starts from there, so every worktree has one; `lyxcwd.resolveCore` reads the worktree's own marker, and everything that reads `Location.AnchorRel` follows.
- When a weft branch merges into its parent's weft at landing, the parent keeps its own `.lyx-anchor`, kept out of the merge as raddle is.
- Creating a beam takes an option for a new anchor (such as `lyx beam create --anchor <dir>`); the default is the anchor of the worktree it is spawned from, which becomes its parent.

## Where the work is

- The marker read: `lyxcwd`, and `fabricengine/codebinding.go`'s own read of the marker.
- The marker writers: `fabric clone`, `reconcile`, `unwire`, and `mergepaths` keeping the marker out of a landing merge.
- Code that builds another worktree's paths from its own anchor: `fabricengine/warplayout.go`, `pairteardown/teardown.go`, `fabricengine/forkrecords.go`; each must resolve that worktree's own anchor.
- The one-anchor keys of `_portals/`, `_launchers/`, weft pathspecs and the hub and beam workspace files.

## Changing a worktree's anchor (in scope, operator, 2026-10-25)

A verb changes a worktree's anchor: remove its weft junctions, move its files in the weft to the new anchor and commit the new marker, recreate the junctions at the new place in the warp, and regenerate the workspace files that list it.
Moving the files covers the warp's machine-local `<anchor>/.lyx/` as well.
No run may be in flight in the worktree while it moves.
A worktree that already has child worktrees is the messy case, since the children keep the old anchor.

## Run (operator, 2026-10-25)

One loom task with profile `solid`, the anchor-change verb included, since it is tied closely to the rest.
It runs alone, after `adopt-runs-tests` has landed; a run that goes wrong is rolled back through git, since it runs in its own worktree.
