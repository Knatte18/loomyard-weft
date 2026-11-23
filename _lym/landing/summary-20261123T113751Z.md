# Board view keeps a done project, so its members stay under it

The board viewer, `board.html`, filed every member of a done project under the root project `P0`.
A beam's landed task showed under `P0` instead of under its beam once the beam project was done,
and it stayed there until a prune removed it.

## Cause

`board-data.js` carries a `projects` list and gives each entry the handle of its own project.
`viewProjects` in `internal/boardengine/view.go` put only open projects in that list.
The viewer looks each entry's project handle up in the list and falls back to the root when it finds none,
so an entry whose project is done always landed under `P0`.
The README and the project files were right: a done project keeps its own `project-<slug>.md` with its members.

## Change

- `viewProjects` now lists every done project after the open ones, in README order.
  A done project's `parent` is the project whose Done section lists it, or the root when it stands under none.
  The open projects, and their parents, are unchanged:
  an open project under a done one still stands under the root, as in the README.
- The viewer needs no change.
  Its tree is built from `projects`, so a done project's members, done or open, now stand under it,
  and a done project with no entry the filters keep shows nothing, as any empty project does.
- `TestRenderBoardData` gets a done member of the fixture's done project.
  Its project-list subtest now expects the done project, so it fails on the merge base.

## Design decisions

- The issue names `lym board list --text`, but that listing is flat: it shows no project for any row, so nothing in it stands under `P0` but the row order.
  The misplacement under `P0` is the viewer's, which the issue's title names, and the fix is made there.
  The `--text` listing is left as it is.
- A done project shows in the viewer with the same heading as an open one.
  Its entries sit in their Done section, which already says what state they are in.

## Merge danger

**Door:** two-way
**Blast radius:** board
