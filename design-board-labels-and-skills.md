# Label descriptions, board CLI gaps, CLI tree exceptions

`board-labels-and-skills` · Task · bug, enhancement, board, config, selfreport · running · Plan-Bouncer

# Goal

Board labels say what they cover, the board CLI does what the hub needs without hand-built JSON, and the CLI tree walk has no recorded exceptions left.

Labels principle (operator, 2026-10-05): labels exist only to group entries for merging into tasks; they name the lyx module an entry mainly touches, and an entry that belongs to no module carries none.

# Scope change (operator, 2026-10-05)

This task was narrowed while its Discussion was paused.
The skill and stencil work it first carried (ly-drive as a stencil, imperative skill loading) moved to task `lyx-skills-and-stencils`, which waits for two in-flight tasks that edit the same stencils.
The board CLI gaps below were added.
The `ly:board` skill stays; whether it is the only skill is no longer a goal of this task.

## Board labels carry a description of what each covers

From note `board-label-descriptions`.

Choosing the label is not always obvious from the name alone: landing (Describe, Publish, PR-Gate, Finalize) is part of `loom`, Discussion and Plan are `loom`, the batcher is `webster`, glyphs are `quarry`.
A YAML comment cannot carry that, since agents read the labels as a parsed list and `lyx config --set` rewrites the file.

- `labels` in `_lyx/config/board.yaml` becomes a map from label to a one-line description of what it covers.
  The type labels (`types`) get descriptions the same way; they are stored like other labels and only rendered differently.
- Validation keeps checking the keys only.
- A `lyx board labels` verb prints each label with its description.
- The `ly:board` skill tells the caller to read `lyx board labels` before choosing labels.

## Board CLI gaps the hub met on 2026-10-05

Merging notes into tasks forced the hub to extract and rebuild entry JSON with a script.

- `lyx board get` can print an entry's body alone, raw markdown, so a caller can edit it as a file.
- `lyx board merge` takes `--body-file` like `upsert`.
- A merge carries over the `issues` of the entries it removes onto the upserted entry, so GitHub provenance survives folding notes into a task.
- The help of every verb that takes a slug states the slug length limit (32 characters); the hub learned it from a refused intake import.

## Five CLI invariant exceptions in board notes, config, selfreport

From note `cli-tree-exceptions` (GitHub #349).

Part 2 of the test-suite cleanup (#347) added a CLI tree walk that checks every verb against the CLI invariant.
It records five exceptions in `cliTreeFindings` that pin existing behavior which breaks the invariant:

- `lyx board notes` (bare) and `lyx board notes bogus` print an extra "not initialized here" envelope.
- `lyx config` (bare) and `lyx config bogus` refuse with "not a git repository" before listing subcommands or refusing the unknown one.
- `lyx selfreport bogus` exits 0 and prints help instead of refusing the unknown subcommand.

Fix each verb and remove its entry from `cliTreeFindings`, so the walk enforces the invariant for it.
