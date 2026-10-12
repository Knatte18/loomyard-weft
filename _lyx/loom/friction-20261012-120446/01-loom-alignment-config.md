# Batch 01: alignment config, peers verb, alignment package

Card 2's gate and the batch gate both name `./internal/fabricengine_test`, which is not a directory: the external test package lives inside `internal/fabricengine`.
`lyx gate test` reports `[setup failed]` for that path on every run, though every real package passes.
The plan's gate lines for the fabric cards should name `./internal/fabricengine` alone.

Card 3 asks for `lyx loom peers` as a runnable group carrying `AudienceRole`, but `cmd/lyx/clitree_test.go` requires a bare group to print help (exit 0) and a mistyped subcommand to refuse with "unknown subcommand", and `clihelp.Invocable` treats a runnable group with no `Use` token or flag as non-invocable, so it must carry no audience.
The group therefore lists the peers when run inside a lyx worktree and prints its help outside one, and carries no audience annotation; `peers show` carries `AudienceRole`.
