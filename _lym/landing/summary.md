# Board view keeps a done project's members under it

The board viewer filed a done entry under P0 when its project was itself done, and so no longer open.
`viewProjects` in `internal/boardengine/view.go` listed only the root and the open projects.
The page found no project for such an entry and fell back to the root.
The entry now stays under its own project until it is pruned.

`viewProjects` lists the root, then every open project, then every done project, each in README order.
A done project carries the handle of the project whose Done section lists it, so a done project nested in another done one stands under it.
`ViewProject` gains a `done` field in the `board-data.js` projects list.
`board.html` uses it to leave a done project out of the project dropdown, because a done project is not a place to write a new entry.

Closes #188

## Design decisions

- The task fixes the board viewer only.
  The `lym board list --text` listing in `internal/boardcli/text.go` stays as it is.
  That listing is flat and shows no project for any row, and it orders every done entry after the P0 row whatever its project.
  Grouping its rows under projects would be a new listing design, which the board entry does not specify.
- The open projects keep the order and parent they had, so a view with no done project is unchanged apart from the new `done` field.

## To check

- Open the board page on a board that holds a done beam project with done members, and confirm they stand under that project and not under P0.
- Open the new-entry form and confirm the project dropdown lists no done project.
- No test covers `board.html`; the dropdown filter is checked by hand.
  `TestRenderBoardData` in `internal/boardengine/view_test.go` covers the projects list, including a done project nested in a done one.

## Merge danger

**Door:** two-way
**Blast radius:** board
