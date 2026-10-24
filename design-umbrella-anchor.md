# [WS 007] An anchor per umbrella

`umbrella-anchor` · Note · enhancement, fabric, config, workspaces

## Seen (2026-10-18)

The anchor (`fabric clone --subpath`) is set once per hub and decides where `_lyx`, `.lyx/` and config sit in every worktree of the hub.

## Idea

Let an umbrella (`orch-any-worktree`) have an anchor of its own, a different subdirectory of the repository, inherited by every task under it as it inherits the branch.
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
