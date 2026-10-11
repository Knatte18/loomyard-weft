# loom step help contradicts itself and hides the driverless seed

`loom-step-help-seed` · Note · bug, loom

## Seen (2026-10-11, discussion bench)

`lyx loom step --help` contradicts itself: `internal/loomcli/cli.go:313` says step bootstraps the worktree's loom task exactly as `lyx loom start` does, and `:329-330` says it never writes a seed and refuses when none exists.
It refuses on an unseeded pair and points to `lyx loom start`, which spawns an LLM driver.
The bench agent took that route and ended up with an unwanted driver; stepping without a driver needs `lyx shed seed self --recipe loom --driver go --param parent=main` first, which the help does not mention.

## Idea

Make the help say only what step does (refuses without a seed), and name the driverless seed command beside `lyx loom start` in the help and in the refusal text.
