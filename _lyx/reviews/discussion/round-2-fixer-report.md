# Fixer report: discussion, round 2

All three findings were validated against the code and fixed in `_lyx/discussion/decision-record.md`, with the support log updated to match.

## Fixed

- **F1 (BLOCKING, design)**: premise confirmed in `cmd/lyx/clitree_test.go` (`walkCommands` skips `help` by name; `bareProblems` and `bogusProblems` require child listing and `unknown subcommand`).
  D1 now decides that `help` is a runnable group with `Use` `help [<command>...]`; bare `lyx help` keeps the root's help, so `help` is exempt by name from the bare-group check alone, and it passes the unknown-subcommand, audience and form tests, with `help` and `index` both `operator`.
  D6 gains the `[<arg>...]` form the `Use` needs, and the form-test acceptance criterion lists it.
  A new acceptance criterion pins the walk's treatment of `help`; the Notes entry on `walkCommands` says the walker's skip goes and the bare check's caller exempts `help`.
- **F2 (BLOCKING, design)**: premise confirmed in `internal/orchcli/spawn.go` (`spawnWatcherProcess` starts `os.Executable()` with the start process's environment) and `internal/reedengine/panebin.go` (the prelude is pane-only).
  D4 now names the binary as the absolute path of the binary that ran `lyx orch start` or `--adopt`, the one the pane prelude exports as `LYX_BIN` and the watcher is started from; the watcher's source runs `<path> help index` at each render, never a `PATH` lookup.
  Bound added: a missing path or failed run fails the render and the watcher goes idle with the reason.
  A9, the `PATTERN-spawn-observability` constraint line, a new `PATTERN-pane-binary-resolution` constraint line, the watcher acceptance criterion and a Notes entry are updated; the support log gains the rejected `PATH`-lookup alternative and a revised Q14.
- **F3 (LOW, decision)**: D1 now resolves a topic word by word and refuses the first word that names no child, so `lyx help board bogus` is refused like `lyx help no-such-topic`; the acceptance criterion covers both.
  The support log records the rejected `Find` fallback and Q15.

## Disputed

None.

## Deferred

None.
