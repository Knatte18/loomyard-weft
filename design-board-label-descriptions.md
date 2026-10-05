# Board labels carry a description of what each covers

`board-label-descriptions` · Note · enhancement, board

Board labels exist only to group entries that belong together, so suitable notes can be merged into one task.
They name the lyx module an entry mainly touches; an entry that belongs to no module carries no label.

Choosing the label is not always obvious from the name alone: landing (Describe, Publish, PR-Gate, Finalize) is part of `loom`, Discussion and Plan are `loom`, the batcher is `webster`, glyphs are `quarry`.
A YAML comment cannot carry that, since agents read the labels as a parsed list and `lyx config --set` rewrites the file.

- `labels` in `_lyx/config/board.yaml` becomes a map from label to a one-line description of what it covers.
- Validation keeps checking the keys only.
- A `lyx board labels` verb prints each label with its description.
- The `ly:board` skill tells the caller to read `lyx board labels` before choosing labels.
