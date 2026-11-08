# Beam reed session gets the beam button group

A beam's reed session showed the task buttons (HALT, CANCEL, code, terminal, agent) instead of the beam group.
The distill, refresh, merge-in and land buttons were missing.
`hubgeom` read a pair as a beam only from the beam mark on the pair's origin record.
A beam's pair is made without that mark, and its pawl run sets the mark later.
A session booted in between, such as a beam already running under batten, was classified as a task.

`buttonContext` in `internal/hubgeom/buttons.go` now asks the new `pairIsBeam`.
A pair is a beam when its origin record carries the beam mark.
It is also a beam when the local board's entry for the pair's slug is a project with the beam flag.
`internal/hubgeom/doc.go` describes the second rule.

Closes #85

## Design decisions

- The board is read only after the origin record says "not a beam", so a marked pair costs no board read.
- The board flag counts only on a `project` entry.
  A beam flag on any other kind is ignored, and a test row pins this against a hand-edited board.
- A missing board entry for the slug reads as a task.
  An unreadable origin record or board is an error, as the origin read already was.
- The board is read when reed boots or attaches, since `buttonContext` is called lazily at those points only.

## Tests

`TestReedGeometry_ButtonContext` gains rows for the board beam flag before the mark,
a project without the flag, a non-project entry with the flag, a beam of another slug,
and an unparseable board.

Nothing was left to check or run by hand.
