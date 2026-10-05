# Label descriptions, CLI tree exceptions, ly-drive as a stencil

`board-labels-and-skills` · Task · bug, enhancement, board, config, selfreport, shed, stencil, orch · running · Loom-Preflight

# Goal

Board labels say what they cover, the CLI tree walk has no recorded exceptions left, and the `ly` plugin keeps `ly:board` as its only skill.
`board-label-descriptions` and `ly-drive-as-stencil` both edit `plugins/ly`.
Every lyx-rendered Claude prompt also opens with an imperative to load `scribe:conversation` and `scribe:prose` (`stencil-loads-skills`, operator 2026-10-05), which touches the same stencils as `ly-drive-as-stencil`.

Labels principle (operator, 2026-10-05): labels exist only to group entries for merging into tasks; they name the lyx module an entry mainly touches, and an entry that belongs to no module carries none.


## Board labels carry a description of what each covers

From note `board-label-descriptions`.

Board labels exist only to group entries that belong together, so suitable notes can be merged into one task.
They name the lyx module an entry mainly touches; an entry that belongs to no module carries no label.

Choosing the label is not always obvious from the name alone: landing (Describe, Publish, PR-Gate, Finalize) is part of `loom`, Discussion and Plan are `loom`, the batcher is `webster`, glyphs are `quarry`.
A YAML comment cannot carry that, since agents read the labels as a parsed list and `lyx config --set` rewrites the file.

- `labels` in `_lyx/config/board.yaml` becomes a map from label to a one-line description of what it covers.
- Validation keeps checking the keys only.
- A `lyx board labels` verb prints each label with its description.
- The `ly:board` skill tells the caller to read `lyx board labels` before choosing labels.

## Five CLI invariant exceptions in board notes, config, selfreport

From note `cli-tree-exceptions`.

Part 2 of the test-suite cleanup (#347) added a CLI tree walk that checks every verb against the CLI invariant.
It records five exceptions in `cliTreeFindings` that pin existing behavior which breaks the invariant:

- `lyx board notes` (bare) and `lyx board notes bogus` print an extra "not initialized here" envelope.
- `lyx config` (bare) and `lyx config bogus` refuse with "not a git repository" before listing subcommands or refusing the unknown one.
- `lyx selfreport bogus` exits 0 and prints help instead of refusing the unknown subcommand.

Fix each verb and remove its entry from `cliTreeFindings`, so the walk enforces the invariant for it.


From GitHub #349 (intake 2026-10-04).

## The ly-drive skill becomes a stencil prompt

From note `ly-drive-as-stencil`.

The `ly:ly-drive` skill exists only to start the drivers that run a loom through Shed with repeated `lyx shed step`.
Nothing about that needs to be a skill: every other Claude process lyx spawns gets its instructions from a stencil prompt, and the driver should too.

- Move ly-drive's content into a stencil under `contracts/stencils/` and have the launch prompt render it.
- Remove the `ly-drive` skill from the `ly` plugin.

`ly:board` stays a skill: it serves the operator's own sessions and the orchestrator, not a spawned process, and it becomes the only skill Loomyard ships.

## Stencils make sessions load scribe:conversation and scribe:prose

From note `stencil-loads-skills`.

The scribe plugin's `SessionStart` hook asks a session to load `scribe:conversation` and `scribe:prose`, but a hook can only ask: on 2026-10-05 the orch session saw the request and did not load either skill.

Idea (operator): lyx makes the loading imperative itself.
The stencils lyx renders for its Claude sessions open with an injected instruction to load both skills before anything else, so no session depends on the plugin hook being obeyed.

- The orch prompt carries it first.
- Settle which spawned roles need it too: every role that writes prose for the operator or into the repo.
