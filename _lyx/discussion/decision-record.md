# Trace-log retention: trace-aware, activity-ranked, configurable

## Goal

Stop routine lyx invocations from evicting the trace files a loom run or an ly-drive step depends on.
After a shed step returns, its `trace_file` must outrank every trace whose last activity is older,
so it survives until N newer-active traces exist or the age bound passes, rather than being the first file the next sweep deletes;
the operator must be able to widen the retention window without a code change.

## Scope

In scope:

- `internal/logger`'s retention sweep (`retention.go`) and the point in the process lifecycle at which it runs (`sink.go`, `cmd/lyx/main.go`'s exit path).
- A new module config file, `_lyx/config/logger.yaml`, carrying the two retention bounds, with its embedded template, reconcile registration, and the CLI-layer code that loads it and hands the bounds to the logger.
- Docs: the logger package doc comment's retention section, `docs/shared-libs/README.md`'s logger line, and `CONSTRAINTS.md`'s Config Strictness Invariant degrading-set list.

Out of scope:

- Which records reach the durable sink, and when the sink arms.
  The two arm triggers (first Info+ record, non-zero exit) and the level policy stay as they are.
- The per-file 8 MiB size cap and its truncation marker.
- `internal/reedengine`'s own server-log pruning under `_board/.lyx/logs`, which is a separate mechanism.
- Propagating `LYX_TRACE_ID` into tmux-spawned agent sessions.

## Decisions

### D1 — The count bound counts traces, not files

Decision: `Sweep` groups the trace files in a directory by the trace id in their filename.
The count bound keeps the newest N groups and deletes every file of each older group.
A group with at least one file owned by a live pid (including the sweeping process) is exempt from deletion and does not consume count budget.
Files that do not match the trace-file grammar stay untouched, as today.

Rationale: a trace id names one invocation tree — a process plus every child that adopted its exported `LYX_TRACE_ID`.
Counting files lets a single invocation whose direct-exec lyx children inherit its exported trace id consume most of the budget;
tmux-spawned agent sessions have `LYX_TRACE_ID` stripped, so their lyx calls form their own groups, and D2 covers that case.
Deleting half of a tree leaves a trace nobody can reconstruct.
Exempting a whole live group extends the existing live-pid guarantee to the children of a running invocation.

### D2 — Rank and age by last activity (newest mtime), not by filename timestamp

Decision: a group's activity time is the newest modification time among its files, falling back to the filename timestamp for any file whose stat fails.
Groups are ranked by activity time for the count bound;
the age bound deletes a non-live group whose activity time is older than the age bound.
This replaces the current "ranked by filename timestamp, never mtime" rule.

Rationale: a filename timestamp is the moment a process first armed its sink.
A long step (Webster, ~tens of minutes) arms at its start, then every lyx child it spawns without an inherited trace id arms later and outranks it;
the moment the step exits and loses its live-pid exemption, the next sweep deletes it as the oldest.
That matches the `operator-surface` observation: the driver's `trace_file` named a missing file.
The step's final `shed: step done` record gives its file the newest mtime at exit, so activity ranking keeps it.

### D3 — The sweep runs at process exit, not at sink arm

Decision: the sweep moves out of `armDurableSinkLocked` and into the exit path.
At exit, a process sweeps its sink directory if and only if its durable sink armed during the process (current generation), including an arm forced by a non-zero exit code.
A process that never armed a sink sweeps nothing.
The sweep runs outside `sinkMu`.

The exit path runs in this order:

1. `NotifyExit(code)`, which force-arms the sink on a non-zero code, as today.
2. The CLI layer asks the logger whether the sink armed in the current generation, and whether it was redirected.
3. Not armed: stop — no config load, no sweep.
4. Armed and redirected: sweep with the template defaults, without loading `logger.yaml`.
5. Armed and not redirected: load `logger.yaml` (D4, D5), then sweep with the resulting bounds.

The config load never runs before step 2, and never in a process whose sink has not armed.
`configengine.LoadOrTemplate` logs at Info when it degrades to the template, and an Info record arms the sink,
so a load in a process that never armed would arm it and write a trace file.

Rationale: loading the config in the CLI layer at exit lets the sweep use configured bounds without the logger importing `internal/configengine` (which imports `internal/logger` — a cycle).
Loading it only after the arm check keeps the arm triggers unchanged: the load's own logging lands in a trace file that already exists.
Sweeping after the load, outside `sinkMu`, also means the sweep never runs while the config load itself is logging, so it cannot deadlock on the non-reentrant `sinkMu`.
A process killed before exit skips its sweep;
the next exiting process sweeps the same directory, so the bound still converges.

### D4 — Both bounds are config keys in `_lyx/config/logger.yaml`

Decision: a new module config `logger.yaml` with two keys:

- `trace_retention_count` — number of traces to keep (default `200`).
- `trace_retention_days` — age bound in days (default `14`).

Both must be positive integers.
The file is loaded with `configengine.LoadOrTemplate` (degrading), so a worktree without `_lyx/` or a hub/standalone state directory uses the template defaults.
The config is resolved from the same lyx-owned worktree anchor the cwd-anchored sink resolves from;
a process whose sink was redirected (`SetDurableSinkDir*`: reed watchdog, standalone wiring) uses the template defaults.
The logger exposes a setter or an exit-time parameter for the bounds, and a query for whether its sink armed in the current generation and was redirected;
the CLI layer (cmd/lyx root or a cliwire helper — the plan picks the site) loads the config in the order D3 fixes and passes the bounds in.
The logger keeps compiled-in defaults equal to the template values for any process that never passes bounds.

Rationale: every process sweeping one directory reads the same per-worktree file, so they agree on the bounds, which environment variables cannot guarantee across tmux-spawned children.
The count default rises from 50 to 200 because the main worktree's 50 slots turned over in about 25 minutes of routine board traffic (observed in `loomyard/.lyx/logs`);
trace files for routine commands are about 1 KB each, so 200 costs negligible disk.
The age default stays 14 days.

### D5 — An unusable retention config never fails the command

Decision: if loading `logger.yaml` fails (malformed YAML, missing key, non-positive or non-integer value), the exit sweep uses the compiled-in defaults and emits one Warn record naming the file and the error.
The process exit code is unchanged.

Rationale: retention is housekeeping at exit;
turning a bad config into a failed `lyx board upsert` would punish an unrelated command.
D3 loads the config only in a process whose sink already armed, so the Warn lands in that process's trace file and the misconfiguration is discoverable.

## Constraints

- `internal/logger` must not import `internal/configengine` (`configengine` imports `logger`).
- `sinkMu` is non-reentrant: nothing that takes it may run config loading or logging while holding it.
- The retention config load adds no arm trigger: it never runs in a process whose sink has not armed (D3).
- Config Strictness Invariant: `logger.yaml`'s loader adopts `LoadOrTemplate` only, and `CONSTRAINTS.md`'s degrading-set list gains the owning module in the same commit.
- Cwd Resolution Invariant: config and sink directory resolution go through `internal/lyxcwd`;
  no new cwd resolution.
- Durable-vs-Ephemeral State Invariant: config lives under `_lyx/config/`, trace files stay under `.lyx/logs`.
- The `go test` gate stays: under `testing.Testing()` without `LYX_TRACE=1` no sink arms, so no exit sweep runs.
- Delete failures stay silently tolerated;
  an absent or empty directory stays a no-op.
- Documentation Lifecycle: the logger package doc, `docs/shared-libs/README.md`, and `CONSTRAINTS.md` change in the same commit as the code.

## Auto-mode assumptions

- The config file is named after the module that owns the behaviour (`logger`), matching the one-file-per-module pattern of the existing `_lyx/config/*.yaml`.
- Key names `trace_retention_count` / `trace_retention_days` are chosen here; the operator did not name them.
- Default count 200 is a judgment from observed turnover, not an operator figure.
- Days, not a Go duration string, as the age unit: matches the existing `*_timeout_min` integer style in `loom.yaml`.

## Open risks

- The `operator-surface` failure was not reproduced.
  If its cause was not retention (for example `.lyx` junction rewiring moving the logs directory mid-step), D1–D3 do not fix it.
  The acceptance test for D2 pins the retention path; any other cause stays open.
- mtime is writable by any process and can be reset by tools that copy without preserving it.
  A copied trace directory ranks by copy time; the fallback to the filename timestamp covers only a failed stat.
- Sweep-at-exit means a long-lived daemon (reed watchdog) sweeps its directory only when it exits;
  other processes that arm a sink in the same directory keep it bounded meanwhile.

## Acceptance criteria

- `Sweep` given files from several trace ids keeps the newest N traces by activity time and removes every file of each older trace; a trace with one live-pid file keeps all its files and does not count toward N.
- A trace whose filename timestamp is oldest but whose mtime is newest survives a count-bound sweep (the Webster-step scenario: step file armed first, N+ later child/routine traces, step file written last).
- A non-live trace whose newest mtime is older than the age bound is deleted even when the count bound would keep it;
  a live trace older than the age bound is kept.
- A file whose stat fails falls back to its filename timestamp and neither panics nor aborts the sweep.
- Non-matching filenames and subdirectories are untouched; an absent or empty directory returns nil.
- The sink arm no longer sweeps; a process that armed its sink sweeps its sink directory exactly once at exit (zero and non-zero exit codes); a process that never armed sweeps nothing.
- The exit sweep uses bounds from `_lyx/config/logger.yaml` when present and valid, template defaults when `_lyx/` is absent or the sink was redirected, and compiled-in defaults plus one Warn record when the file is invalid — with the exit code unchanged in every case.
- A process that never armed its sink, run in a lyx worktree whose `logger.yaml` is absent or invalid, writes no trace file and loads no config at exit.
- `logger.yaml` rejects zero, negative, and non-integer values for both keys.
- `lyx config reconcile` recognises `logger.yaml` and seeds it from the template.
- Existing retention tests (`retention_test.go`) are updated to the trace-group and activity-time semantics, not deleted.
- `go test ./...` and the repo's help-tree / constraint tests pass.

## Notes for the plan writer

- `Sweep(dir)` today is called from `armDurableSinkLocked` in `internal/logger/sink.go`; `NotifyExit(code)` in the same file is the existing exit hook, called from `cmd/lyx/main.go`'s `main` and `run`.
- `internal/shedverbs/status.go` lists trace files by trace id through `logger.TraceDir()`; grouping by trace id in retention aligns with that consumer.
- `internal/logger/logger.go`'s package doc has a "The durable sink and its retention" section that states the current bounds and the live-pid rule; it must describe the new semantics.
- `internal/shuttleengine/wait.go` documents the trace filename shape; the filename grammar does not change.
