# Five CLI invariant exceptions in board notes, config, selfreport

`cli-tree-exceptions` · Note · bug, board, config, selfreport

Part 2 of the test-suite cleanup (#347) added a CLI tree walk that checks every verb against the CLI invariant.
It records five exceptions in `cliTreeFindings` that pin existing behavior which breaks the invariant:

- `lyx board notes` (bare) and `lyx board notes bogus` print an extra "not initialized here" envelope.
- `lyx config` (bare) and `lyx config bogus` refuse with "not a git repository" before listing subcommands or refusing the unknown one.
- `lyx selfreport bogus` exits 0 and prints help instead of refusing the unknown subcommand.

Fix each verb and remove its entry from `cliTreeFindings`, so the walk enforces the invariant for it.


From GitHub #349 (intake 2026-10-04).
