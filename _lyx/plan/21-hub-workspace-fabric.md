# Card 21 — hub-workspace-fabric

**Edit:**
- `internal/fabricengine/hubworkspace.go`
- `internal/fabricengine/hubworkspace_test.go`
- `internal/fabricengine/portals.go`
- `internal/fabricengine/hubworkspace_write.go`
- `internal/fabricengine/portallauncher_test.go`

**Intent:** Close the two fabric-side hub-workspace follow-ups from the PR #301 review (D13, first and third bullets).

`HubWorkspaceFolders` builds the prime's folder path from `WorktreePath(l, primeName)` joined with `l.AnchorRel`, instead of joining `l.HubPath` and `primeName` by hand.

`removePortal` stops pruning at the anchor directory `_portals/<AnchorRel>`, not at `_portals`.
Only the pair's own entry goes; the emptied anchor directory stays, lyx never removes it, and a later pair under it reuses it.
Any other caller reaching the same prune for portals (the dead-slug teardown in `prune.go` goes through `removePortal`; confirm) inherits the bound.
The launcher-side prune is unchanged.
`hubworkspace_write.go`'s comment that says removing the last pair prunes the anchor directory is corrected.

`HubWorkspaceFolders` returns existing entries only: a folder whose target directory does not exist is left out.
The prime and `_board` exist on a live hub, so in practice this drops a never-created `_portals/<anchor>`.
A stat error other than not-exist is returned, not swallowed.

Tests:
- `hubworkspace_test.go` asserts the prime folder path equals the `WorktreePath`-derived one on a subpath-anchored location.
- It asserts a missing portal anchor directory is omitted from the list while present ones stay in order.
- `portallauncher_test.go` (or the portal test that already removes a last pair) asserts that removing the last pair leaves `_portals/<anchor>` in place and empty, and that a pair added afterwards lands under it.

**ImpactSummary:** HubWorkspaceFolders gains an existence filter and one path-construction change; portal removal stops one level higher; only ideengine consumes the folder list.
