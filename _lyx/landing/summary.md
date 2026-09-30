# logger: activity-ranked, configurable trace retention at exit

Routine lyx invocations (board upserts, fabric verbs) used to fill the 50-file trace budget within minutes.
Retention also ranked files by the timestamp in the filename, so a long shed step armed first and was the oldest file the moment it lost its live-pid exemption.
The driver's `trace_file` then named a missing file, as seen in run `operator-surface`.

This change makes retention trace-aware, activity-ranked and configurable, and moves the sweep to process exit.

## What changes

- `internal/logger` `Sweep` groups trace files by trace id.
  The count bound keeps the newest N traces and deletes every file of each older one.
  A trace with any live-pid file (including the sweeping process) is kept whole and does not consume count budget.
- A trace's activity time is the newest mtime among its files, falling back to the filename timestamp when a stat fails.
  Both the count ranking and the age bound use it.
  The step's final record therefore keeps its file among the newest.
- The sweep no longer runs when the sink arms.
  A process whose sink armed during its run (including an arm forced by a non-zero exit) sweeps its sink directory once at exit.
  A process that never armed sweeps nothing and loads no config.
- New `_lyx/config/logger.yaml` with `trace_retention_count` (default 200) and `trace_retention_days` (default 14).
  Both must be positive integers.
  It is registered with `configreg`, so `lyx config reconcile` seeds it from the template.
- Docs: the logger package doc's retention section, `docs/shared-libs/README.md` and the degrading-set list in `CONSTRAINTS.md`.

## Design decisions

- **Config loads in the CLI layer, at exit** (`cmd/lyx/exitsweep.go`).
  `internal/logger` cannot import `internal/configengine`, which imports it.
  The logger exposes whether its sink armed in the current generation and whether it was redirected, and accepts the bounds.
- **The load runs only after the arm check.**
  `LoadOrTemplate` logs at Info when it degrades, and an Info record arms the sink, so loading in a never-armed process would write a trace file.
  The sweep runs outside `sinkMu`.
- **Redirected sinks use the template defaults** without loading `logger.yaml` (reed watchdog, standalone wiring).
- **A bad `logger.yaml` never fails the command.**
  The sweep falls back to compiled-in defaults and emits one Warn naming the file and the error; the exit code is unchanged.
- `loggerconfig` also rejects `trace_retention_days` values beyond the `time.Duration` range.
- Count default rises from 50 to 200; routine trace files are about 1 KB, so the disk cost is negligible.
- Days rather than a duration string, matching the `*_timeout_min` integer style in `loom.yaml`.

## Risks and limits

- The `operator-surface` failure was not reproduced.
  If its cause was something other than retention (for example `.lyx` junction rewiring), this change does not fix it; the tests pin only the retention path.
- mtime can be reset by tools that copy without preserving it; a copied trace directory ranks by copy time.
- A long-lived daemon such as the reed watchdog sweeps only when it exits; other processes arming a sink in the same directory keep it bounded meanwhile.
- A process killed before exit skips its sweep; the next exiting process converges the directory.

## To check by hand

- Not run: a real loom run or `ly-drive` step end to end, confirming the step's `trace_file` still exists after its child lyx calls exit.
- Not run: `lyx config reconcile` against a real worktree to see `logger.yaml` seeded.
