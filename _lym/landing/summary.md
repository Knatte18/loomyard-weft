# raddle: file each part with its own module labels

Raddle-File used to copy the Sketch's whole module set onto every entry it filed.
A part that changed one package carried every module label of its Sketch, so a label filter such as `--label reed` matched parts that never touch reed.
The board listing also ran very wide.

Now the split proposal names each part's module labels, and Raddle-File files the part with its type label and those labels.

## What changes

- `raddleparser`: `Part` gains a `labels` field, the module labels of the modules the part's own work changes.
  The proposal check gains three findings:
  - `labels-missing`: a part with no module label.
  - `label-unknown`: a part label that is not a module label of the board's vocabulary; a type label counts as unknown here.
  - `label-duplicate`: a part that lists one module label more than once, one finding per repeated label.
    The board refuses an entry that carries a label twice, so the check catches it before filing.
- A part label the Sketch does not carry is accepted, since a split can find a module the Sketch's labels missed.
- `BoardContext` gains `LabelVocabulary`, the board's configured labels that are not type labels.
  `loomcli`'s `raddlecontext.go` fills it, and `boardengine/raddlefile.go` follows the new shape.
- `loomshed` `Raddle-File`: a part is filed with its `type` and its own `labels`.
  The project is filed with its `type` and every label its parts carry, each once, in the order the parts first name them.
- The raddle stencil and the raddle review rubric state the rule beside the other part fields, so the split perch weighs each part's labels against the packages its body names.
- `raddle-proposal-spec.md` documents the field and the three checks.

## Design decisions

- The project's labels are the union of its parts' labels rather than the Sketch's set, since a project touches every module its parts change.
- An unknown label is refused rather than filed, so a typo cannot reach the board; the finding's way forward names `lym board labels` and the operator adding a label to the board config.

## To check by hand

- `part-label-duplicate-passes-check`: a review flagged that a part listing one label twice passed the check.
  The `label-duplicate` finding in `internal/raddleparser/check.go` (`vocabulary`) addresses it; confirm it reports each repeated label once.
- No run of the split perch against a real Sketch was made; confirm a fresh proposal lists per-part labels and that the filed entries carry them.
