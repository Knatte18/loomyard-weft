# Plan card spec contradicts CLI tree invariants for a runnable group with an audience

`plan-cli-tree-check` · Note · bug

Imported from [issue #588](https://github.com/Knatte18/loomyard/issues/588).

A card asked for a new `lyx loom peers` verb as a runnable group carrying `AudienceRole`.
`cmd/lyx/clitree_test.go` requires a bare group to print help (exit 0) and a mistyped subcommand to refuse with "unknown subcommand", and `clihelp.Invocable` treats a runnable group with no `Use` token or flag as non-invocable, so such a group must carry no audience.
The implementer had to deviate from the card: the group carries no audience annotation and only the `peers show` subcommand carries `AudienceRole`.

Abnormal: the plan specified a shape the CLI tree invariants forbid, and the conflict surfaced only during implementation.

To do: have plan authoring or plan review check CLI-surface cards against the `clitree_test.go` and `clihelp.Invocable` rules, and document in the CLI-authoring docs that audience annotations belong on leaf verbs.

Provenance: task `alignment-agent`, note `01-loom-alignment-config.md`.
