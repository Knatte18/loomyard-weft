# Webster round 2 — fixer report

## Fixed

- **F1** (BLOCKING, consistency) — commit `41b2215b7`.
  I checked each of the five locations against the diff, and each holds an edited sentence that still broke mid-phrase.
  Each edited sentence now sits on its own line:
  the `ValidateDispatch` doc in `internal/planglyph/planglyph.go`, the `internal/planparser/handle.go` and `handle_test.go` file headers, the `ledger` doc in `internal/planparser/shape.go`, and the `checkGlyphMalformed` doc in `internal/planparser/validate.go`.
  A sentence that began on the same line as an edited one had to move to a new line.
  I put it on one line, or broke it only at a semicolon, as in the `classifyRef` sentence in `handle.go`.
  Untouched sentences that start on their own line keep their wrapping.
- **F2** (NIT, consistency) — commit `b0cd6cb00`.
  `editTarget`'s doc comment in `internal/planglyph/deleteorder.go` now reads "editTarget is one Edit-group ref of a card that is a member glyph or a file, with its shape and disk path."

Neither finding was a coverage gap, so I added no test.
Checks: `go build ./...` and the full `go test ./...` pass, and so does `lyx loom lint-comments --base fd5b0f58e`.

## Disputed

None.

## Deferred

None.
