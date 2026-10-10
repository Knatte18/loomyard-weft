# Config-driven file log level

`logger-file-level` · Note · enhancement, config

## Seen

The trace files written for each lyx invocation hold only high-level messages, and `_lyx/config/logger.yaml` sets only retention (`trace_retention_count`, `trace_retention_days`).
The file log has no configurable level, so after the 2026-10-08 crash the traces could not show what lyx's subcommands did.
Activity-ranked pruning also left the per-minute invocation counts incomplete.

## Idea

`logger.yaml` gains a `level` key (error, warn, info, debug) for the file trace, independent of the `-v` console verbosity, with info as the default.
Every engine logs its decisions and external calls at info or debug, so raising the level makes most of what lyx does traceable without a rebuild.
The spawn audit stays always on, whatever the level, because it cannot be turned on after a crash.
