# An anchor per beam

`umbrella-anchor` · Note · enhancement, fabric, config, workspaces

## Seen (2026-10-18)

The anchor (`fabric clone --subpath`) is set once per hub and decides where `_lyx`, `.lyx/` and config sit in every worktree of the hub.

## Idea

Let an umbrella (a beam, built in dd11ab09f) have an anchor of its own, a different subdirectory of the repository, inherited by every task under it as it inherits the branch.
Beams exist now, and all of them still use the hub's one anchor.
Fabric, batten and loom would then read the anchor from the task's umbrella rather than from the hub.

## Operator view, not a decision (2026-10-24)

Low priority. Every beam has a parent, so a beam could get an anchor different from its parent's, which gets messy: at landing, the beam's `_lyx/` files, decisions and config would sit under a different subdirectory than the parent reads.
One way out (orch suggestion, not decided): only a beam directly under `main` may have its own anchor; a sub-beam always inherits its parent's.

## Effort study (Sonnet agent, 2026-10-24)

Medium-hard, loom-sized, not a darn.
- The read side is central: `lyxcwd.resolveCore` alone reads the hub marker `_board/.lyx-anchor`, and the anchor travels as `Location.AnchorRel` (about 50 non-test uses in about 25 packages).
- Hub-wide assumptions are scattered: `fabricengine/warplayout.go` copies the prime's anchor to siblings, as do `pairteardown/teardown.go` and `fabricengine/forkrecords.go`; weft pathspecs, `_portals/`, `_launchers/` and the hub workspace file are keyed on one anchor; `fabricengine/codebinding.go` reads the marker itself.
- Hardest design point: the beam's anchor cannot live in its origin record under `<anchor>/_lyx/`, since the anchor is needed to find it; it needs an anchor-independent index on Bolt.
- Size: 6-8 packages in substance, about 600-1000 lines of code and 400-800 of tests; safer split into lookup and `Location` plumbing, then pair and weft pathspecs, then portals, launchers and workspace.
- Cheapest together with or just after `umbrella-groundwork`, which changes the same seams (`add.go`, `origin.go`, `warplayout.go`, teardown).

## Why an anchor per worktree (operator, 2026-10-24)

The anchor decides where the weft junctions are made and placed inside the matching warp, and where the hub's and each beam's VS Code workspace file roots a worktree; both let work happen in a deep subdirectory without walking far down to it.
All beams share one hub and one board, so a separate hub per subdirectory (`fabric clone --subpath`, a whole new weft repo and board) is no answer.

## Idea (operator, 2026-10-24): `.lyx-anchor` at each weft branch's root

- Every weft branch may carry a `.lyx-anchor` at its own root, so the anchor is set per beam and in effect per worktree.
- This breaks the circular lookup above: the weft worktree sits at `<hub>/<slug>-weft`, so lyx finds the marker without knowing the anchor.
- Inheritance comes from git: a task's weft branch starts from its beam's weft branch and carries the beam's marker; a worktree with another anchor commits its own.
- No marker stays on Bolt: `fabric clone --subpath` writes the first `.lyx-anchor` at the root of `main-weft`, and every weft branch starts from there, so every worktree has one; `lyxcwd.resolveCore` reads the worktree's own marker, and everything that reads `Location.AnchorRel` follows.
- The work is in the code that builds another worktree's paths from its own anchor (`warplayout.go`, `teardown.go`, `forkrecords.go`), `codebinding.go`'s own marker read, and the one-anchor keys of `_portals/`, `_launchers/` and the workspace files: each must resolve that worktree's own anchor.
- Changing a worktree's anchor becomes a modest verb: remove its weft junctions, move its files in the weft to the new anchor and commit the new marker, recreate the junctions at the new place in the warp, and regenerate the workspace files that list it.
  Moving the files covers the warp's machine-local `<anchor>/.lyx/` as well.
  No run may be in flight in the worktree while it moves (operator, 2026-10-24).
  Operator view, not a decision: changing the anchor of a worktree that already has child worktrees is very messy, since the children keep the old one.
- Creating a beam takes a simple option for a new anchor (such as `lyx beam create --anchor <dir>`); the default is the anchor of the worktree it is spawned from, which becomes its parent (operator, 2026-10-24).
- To check: if a weft branch merges into its parent's weft at landing, the parent keeps its own `.lyx-anchor`, kept out of the merge as raddle is.

## Operator view, not a decision (2026-10-24): one large isolated task

The operator is very sceptical that this works.
When it becomes a task, it is one large, isolated task that preferably runs alone, closely watched, and is rolled back through git if it goes wrong.
