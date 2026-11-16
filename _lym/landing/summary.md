# Board README entry cell shows pull_request and on_land

The board README's entry cell showed a task's brief and its run settings, and nothing else a run reads.
A task with `pull_request` on, such as `adopt-build-revision`, looked like it opens no pull request.

The entry cell now adds one code-span bullet per run field the entry sets, between the brief and the run settings:

- `pull_request: <value>`
- `on_land: <key>`

A field left empty gives no bullet.
The bullets stack in the title cell, so the table gains no column.

## Design decisions

- `runFieldLines` in `internal/boardengine/render.go` builds the lines in a fixed order, `pull_request` first.
- `profile` and `recipe` get no bullet, because the Recipe cell already shows them.
  The package doc in `doc.go` now says the profile sits in the Recipe cell, not the entry cell.
- Run fields were checked against the task's field list.
  `pull_request` and `on_land` are the ones that change how a run goes and that the cell omitted.
- The board.json keys `pull_request` and `on_land` are now the constants `pullRequestKey` and `onLandKey` in `task.go`.
  The renderer, `replay.go`, `store.go` and `front.go` use them.

## Tests

The README golden fixture in `render_test.go` now sets `pull_request` and `on_land` on two tasks.
The golden tables pin the bullet order and their place between the brief and the settings.

## To check by hand

- `run-field-key-literals`: confirm no bare `"pull_request"` or `"on_land"` literal is left in `internal/boardengine` outside `task.go`, tests and doc comments.
  Some keys in the `replay.go` table and the `UpsertPayloadKeys` list use the constants now, but the rest of the package was not swept.
