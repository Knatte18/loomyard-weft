# A plain promote of a Sketch takes recipe raddle

`lym board promote <slug>` of a Sketch now promotes it with recipe `raddle`, as if `--recipe raddle` were given.
Raddle is the one recipe a Sketch may take, so the flag only added typing.
The handle form works too: `lym board promote S3`.

## What changes

- `--recipe raddle` stays accepted.
  Any other recipe on a Sketch is still refused, and the refusal now names the plain promote as the way forward.
- `Store.promoteTo` in `internal/boardengine/store.go` fills in recipe `raddle` when the caller gave none and the entry is a discussed note that holds a stored decision record or carries the sketch label.
  The existing raddle path then keeps the record and flag, restores a dropped sketch label, and releases the task in the same write.
- `boardengine.SketchPromoteCommand` in `internal/boardengine/sketch.go` builds the way-forward command, `lym board promote '{"slug":"<slug>"}'`.
  The refusals in `checkDiscussedPromotion`, `checkSketchLabel` and batten's raddle start guard (`internal/battencli/startguard.go`) all use it, so the text lives in one place.
- The `lym board promote` help and examples, the `internal/boardcli` and `internal/boardengine` package docs, `docs/overview.md` and the `ly:board` skill no longer name `--recipe raddle` as the required form.

## Design decisions

- The default keys on the stored record or the sketch label, not on the label alone.
  A discussed note whose label a note write stripped still has its record, so a plain promote of it takes raddle and restores the label rather than hitting the discussed-promotion refusal.
- A discussed note with no sketch label and no stored record is not judged: a plain promote of it takes no recipe, as before.
- The way-forward command uses the JSON form with a slug, not the bare slug, so it is valid for every slug the board accepts.

## To check by hand

- A note that carries the sketch label but whose record file is missing now takes raddle on a plain promote.
  Check that this is the wanted outcome: the task is made with recipe `raddle` and no stored record, so batten's start guard then refuses the run with its "holds none" message, which names the motif Discussion and a re-promote.
  `PlainPromoteOfASketchWithAMissingRecordFileTakesRaddle` in `internal/boardengine/sketch_test.go` pins the current behaviour.
