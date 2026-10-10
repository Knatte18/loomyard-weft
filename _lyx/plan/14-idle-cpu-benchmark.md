# Card 14 — idle-cpu-benchmark

**Create:**
- `docs/benchmarks/daemon-idle-cpu.md`

**Intent:** The before-and-after idle CPU measurement of `lyx reed watchdog` (D6), a measurement report of one run on one date, with its procedure in the report so the next run repeats it.
The procedure is shell only, no Python: on a hub with at least three idle reed sessions, find the daemon's pid through `pgrep -f "lyx reed watchdog"`, read `utime` and `stime` from `/proc/<pid>/stat` at the start and after ten minutes, and report CPU seconds per hour; the after figure also counts tmux spawns per minute from the hub's durable log at Debug (`LYX_LOG_LEVEL=debug` on the daemon), the `list-sessions`, `list-panes` and `capture-pane` lines of the discovery, resize and name-repair loops.
The before figure is taken against the daemon the production `lyx` runs on the operator's hub, which is `main` before this task lands; the after figure against this branch's `./deploy-dev` build run in the foreground with `--hub-path`, `--tmux` and `--shell` after the production daemon's process is ended, which the next `lyx reed up` or attach respawns.
The report follows `docs/benchmarks/haiku-5-5.md`'s shape: the date and machine, the procedure, one table with both figures and the spawn counts, and one paragraph on what the numbers say about the stat tick against the process spawns.
It pins numbers, which a measurement report may.
