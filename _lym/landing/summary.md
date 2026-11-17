# raddle: each filed part carries its own module labels

Raddle-File filed every part of a split with the Sketch's whole module-label set, whatever packages the part changes.
A label filter such as `--label reed` then matched parts that never touch reed.
Each part now names its own module labels in the proposal, and Raddle-File files those.

## What changes

- **Proposal format.**
  A part gains the `labels` key: the module labels of the modules the part's own work changes (`raddleparser.Part.Labels`).
  `contracts/specs/raddle-proposal-spec.md` documents the key, the two new checks and how Raddle-File files the labels.
- **Checks.**
  `raddleparser.Check` adds `labels-missing`, a part with no module label,
  and `label-unknown`, a part label that is not one of the board's module labels; a type label counts as unknown.
  A label the Sketch does not carry passes, since a split can find a module the Sketch's labels missed.
  The checks read the board config's labels through the new `BoardContext.LabelVocabulary`, which `loomcli`'s `raddleBoardContext` fills.
  Each finding names its way forward.
- **Filing.**
  `loomshed`'s Raddle-File files each part with its type label plus its own labels.
  A project entry gets its type label plus every module label its parts carry, each once, in the order the parts first name them.
  The Sketch's module labels still scope the `board-unchecked` rule, which is unchanged.
- **Stencil and rubric.**
  The raddle agent's stencil tells it to give each part the labels of the modules its packages fall in, not the Sketch's whole set.
  It also points the agent at `lym board labels`.
  The split perch's rubric adds a judgement:
  a part's labels name the module of each package its body names, and no module the part does not change.
  A mismatch is a `design` finding.

## Design decisions

- A project's labels are the union of its parts' labels. Copying the Sketch's set would bring back the over-wide match one level up.
- `label-unknown` checks against the board config's `labels` list only, so a type label in `labels` is refused rather than filed twice.

## Tests

- `raddleparser`'s `TestCheck` gains rows for a part with no label, an unknown label and a type label used as a module label.
  The fixture's second part carries a label the Sketch lacks and still passes.
- `loomshed`'s Raddle-File test gives the run's entry a Sketch-only label, `mod-sketch`.
  It asserts that every filed part carries only its own labels and that a project carries its parts' labels once each.
- `loomcli`'s `TestRaddleBoardContext` writes a board config with a module label and asserts that it reaches `LabelVocabulary`.
