# Dobby holds new run starts while the cgroup task limit nears

With five loom runs and a darn run live, the user slice reached `pids.current 2997` of `pids.max 3000` and every fork failed.
Dobby now checks the cgroup task limit right before it starts a ready task, and starts none while the free tasks under the limit are fewer than a configured margin.
A machine with no readable cgroup v2 limit starts runs as before.

## What changes

- `internal/proc` gains `ReadTaskLimit`, which reads the calling process's cgroup v2 path from `/proc/self/cgroup` and walks from it up to the root of `/sys/fs/cgroup`.
  It reports the level with the fewest free tasks (`pids.max` minus `pids.current`) as current, max and cgroup path.
  A level whose `pids.max` is `max`, or whose files are missing or malformed, sets no limit; with no readable limit anywhere, `found` is false.
- beam.yaml gets `task_headroom`, the fewest free tasks for dobby to start a run.
  The template default is 500, about one run's worth of room; 0 turns the hold off, and a negative value is refused naming the key like the other keys.
- `internal/dobbyengine` `Deps` gets a `TaskLimit` seam, filled with `proc.ReadTaskLimit` by `hubDobbyDeps` in `internal/beamcli`.
  `startReady` reads it before each task's first pass and, while the headroom is below `task_headroom`, starts no task in that tick.
- The dobby log gets one line per hold episode: `hold starts: <current> of <max> tasks in <cgroup>, below task_headroom <margin>` when a start is first held, and `starts resume: ...` when the first start after it is let through.
- `internal/proc/doc.go` and `internal/dobbyengine/doc.go` document the reader and the hold, and the template comment documents the key.

## Design decisions

- **Walk the ancestry, no hard-coded slice.**
  The user slice is the level that ran out, and it is an ancestor of the dobby and of every tmux pane reed starts; the walk finds it without knowing the uid or slice layout, and also catches a tighter limit set lower down.
- **The hold only delays.**
  A held task gets no refused-start record, notice or backoff and is tried again on the next tick; the state record gains nothing.
  Passes of runs already started are never held, and the timer waits at most a minute between ticks.
- **The first held task ends the step for the tick.**
  Later ready tasks are not tried in that tick, since the reading applies to all of them.
- **Not found reads as no hold.**
  An unreadable `/proc/self/cgroup`, a cgroup v1 host, a non-Linux host, a nil seam and `task_headroom: 0` all start runs as before.
- The hold is per dobby: each beam's dobby reads the same limit and holds on its own.
  Out of scope: the dobby view and `lym batten overview` show no reason for a held task, and a hand `lym batten run` start is never held.

## Limits to know

- A run's first pass forks little at its start, so several dobbies, or several tasks in one tick, can pass the check before the earlier starts grow; the margin has to absorb that.
- A host whose limit sits outside the dobby's own ancestry is never held.
- On a machine where `pids.max` is raised well above the default, the hold never triggers unless `task_headroom` is tuned too.

## Not run by hand

The reader is tested over a temp-dir fixture of the cgroup files and the hold through the fake `Deps`; no test reads the host's real cgroup.
To check on a Linux host: run a dobby with `task_headroom` set above the free tasks under the user slice, confirm the `hold starts` line and that a ready task does not start, then lower it and confirm `starts resume` and the start.
