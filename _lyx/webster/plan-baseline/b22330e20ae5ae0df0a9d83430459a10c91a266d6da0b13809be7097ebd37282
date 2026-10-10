# Card 14 — idle-cpu-benchmark

**Create:**
- `docs/benchmarks/daemon-idle-cpu.md`

**Intent:** The before-and-after idle CPU measurement of `lyx reed watchdog` (D6), a measurement report of one run on one date, with its procedure in the report so the next run repeats it.
The procedure is shell only, no Python: on a scratch hub the procedure starts, with at least three idle reed sessions, find the daemon's pid through `pgrep -f "lyx reed watchdog --hub-path <scratch hub>"`, read `utime` and `stime` from `/proc/<pid>/stat` at the start and after ten minutes, and report CPU seconds per hour; the after figure also counts tmux spawns per minute from the hub's durable log at Debug (`LYX_LOG_LEVEL=debug` on the daemon), the `list-sessions`, `list-panes` and `capture-pane` lines of the discovery, resize and name-repair loops.
The scratch hub is cloned into a temporary directory with `lyx fabric clone --into`, and three of its worktrees are brought up with `lyx reed up` and left idle.
Both figures come from that scratch hub, each daemon started in the foreground with `--hub-path`, `--tmux` and `--shell` after the scratch hub's own spawned watchdog is ended: the before figure from a `lyx` built from `main`'s tip before this task lands, extracted with `git archive main` into the scratch directory and built there with `go build` (the production `lyx` is built at `prod`, which can lag `main`), and the after figure from this branch's `./deploy-dev` build; the report names each binary's commit.
No process or file of the operator's own hub is touched, and the scratch hub's sessions are ended with `lyx reed down` after the run.
Bound: the procedure ends only processes of the scratch hub it started, found by the scratch hub's path on their command line.
The report follows `docs/benchmarks/haiku-5-5.md`'s shape: the date and machine, the procedure, one table with both figures and the spawn counts, and one paragraph on what the numbers say about the stat tick against the process spawns.
It pins numbers, which a measurement report may.
