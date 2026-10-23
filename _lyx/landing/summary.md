# Show the prime's owner in the board README and lyx board list

The machine that owns a hub's prime was recorded only as `prime_owner` in `board.json`, so an operator adopting tasks learnt of a foreign owner only from `lyx adopt`'s warning.
The prime's owner now shows wherever task owners do.

## What changes

- The rendered board README states `Prime owner: ` on the line after the render time, with the machine in a code span, or `none recorded` when `board.json` records none.
- `lyx board list` JSON gains a top-level `prime_owner` key beside `tasks`, always present and `""` when none is recorded.
- `lyx board list --text` opens with `prime owner: <machine>`, or `prime owner: none`, followed by the entry lines unchanged.
- `find`, `get` and `list-full` are unchanged; the `find` help now says it prints list's tasks array in an envelope without `prime_owner`.
- The `boardengine` and `boardcli` package docs, the `list` and `find` help and the board skill's list paragraph describe the new output.

## Design decisions

- `Render`, `RenderToDisk` and `renderReadme` take the prime owner as an argument, so the renderer stays a pure function of its inputs.
  Every render site (the board write, the owner setter, the replay) passes its loaded store's recorded owner.
- `Board.PrimeOwner` is a new read that loads the store and returns the recorded owner, `""` for an absent board.
  `list` calls it beside `ListTasksBrief`, which keeps that function's signature and callers untouched.
  The two reads are separate loads, so a write landing between them only shows a fresher owner.
- The shown value is the recorded `prime_owner`, not a resolved one: the prime has no parent to inherit from.
- `board.json`'s shape and the way the owner is recorded and replayed are unchanged.

## Behaviour to know

- A script parsing `list --text` line by line sees one extra first line.
  No consumer in this repository parses it.

## To check by hand

Tests were extended for each point below, but not run by hand against a real board:

- The README's `Prime owner:` line reaches the rendered file from the board write, the owner setter and the replay (`internal/boardengine/board.go`, `ownerset.go`, `replay.go`).
- `lyx board list` against a missing board directory gives an empty `prime_owner`, and `--text` the `none` line (`Board.PrimeOwner` in `internal/boardengine/board.go`).
- The `find` help text in `internal/boardcli/cli.go` matches what `find` prints.
