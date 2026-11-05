# A plain promote of a Sketch takes recipe raddle

`lym board promote <slug>` of a Sketch now makes it a raddle run's entry without a `--recipe` flag.
Raddle is the one recipe a Sketch may take, so the flag added typing and nothing else.

## What changes

- `Store.promoteTo` in `internal/boardengine` gives recipe `raddle` to a promotion with no recipe when the entry is a discussed note whose decision record file is stored.
  The rule keys on the discussed flag and the stored record, as the existing promotion rule does, so a Sketch whose label a note write stripped also takes raddle and gets its label back.
  The promotion then follows the existing raddle path: it keeps the record and flag and releases the task in the same write.
- `--recipe raddle` stays accepted.
  Any other recipe on a Sketch is still refused, and the refusal now names the plain promote as the way forward.
- The handle form works: `lym board promote S3` resolves through the existing handle lookup.
- A discussed note whose record file is missing still promotes like any note, with no recipe set.

## Refusal texts

The new `boardengine.SketchPromoteCommand` builds the way-forward command, and every refusal that leads to the raddle promote uses it: the promotion rule in `sketch.go`, the label rule in `labels.go` and batten's raddle start guard.
The command it names is `lym board promote '{"slug":"<slug>"}'`.
The old texts named `lym board promote <slug> --recipe raddle` with a bare slug, which the verb rejects as invalid JSON, because it takes a JSON payload or a handle.

## Docs

The `promote` help and flag text, the package docs of `boardengine` and `boardcli`, `docs/overview.md` and the `ly:board` skill now describe the plain promote of a Sketch.

## Design decisions

- The refusals name the JSON-slug form instead of a bare slug, since that is the form the verb accepts.
  Accepting a bare slug would change the lookup that every board verb shares, so that stays out of scope.

## Tests

- `TestDiscussedPromotion` now admits a plain promote of a Sketch, of a Sketch by its `S1` handle, and of a note whose label was stripped, each as a released raddle task.
  It still refuses a `darn` promote of a Sketch, and a plain promote of a note whose record file is missing gets no recipe.
- The refusal assertions in `labels_test.go` and batten's `startguard_test.go` expect the new way-forward command.
