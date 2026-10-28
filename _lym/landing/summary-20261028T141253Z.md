# Dobby holds a new run start while the cgroup task limit nears

Before dobby starts a board task, it now reads the cgroup task limit its process sits under, and holds the start while too few tasks are free under it.
With five loom runs and a darn run live, the user slice reached `pids.current 2997` of `pids.max 3000` and every fork failed; this keeps dobby from adding a run at that point.

## What changes

- **`proc.ReadTaskLimit`** (`internal/proc/tasklimit.go`) reads the cgroup v2 path from `/proc/self/cgroup` and walks from that cgroup up to the root of `/sys/fs/cgroup`.
  At each level with a numeric `pids.max` it reads `pids.current`, and returns the level with the fewest free tasks as a `TaskLimit` (current, max, cgroup path).
  A level reading `max`, or with a missing or malformed file, sets no limit; no cgroup v2 line, or no numeric level at all, reads as not found.
- **beam.yaml `task_headroom`**, default 500, is the fewest free tasks under that limit for dobby to start a run; 0 turns the hold off, and a negative value is refused naming the key like the other beam.yaml keys.
- **The hold.** `startReady` calls a new `Deps.TaskLimit` seam right before each start; while the headroom is below `task_headroom`, the tick starts no further task.
  A held task gets no refused-start record, notice or backoff and is tried again on the next tick.
  The dobby log gets `hold starts: <current> of <max> tasks in <cgroup>, below task_headroom <n>` at the first held start of an episode, and `starts resume: <current> of <max> tasks in <cgroup>` (or `starts resume: no task limit applies` when the hold was turned off) at the first start let through after it.
- `hubDobbyDeps` in `internal/beamcli` fills the seam with `proc.ReadTaskLimit`.
- `internal/proc/doc.go` and `internal/dobbyengine/doc.go` describe the reader and the hold, and the beam.yaml template comments the new key.

## Design decisions

- The limit is the tightest level of the dobby's own cgroup ancestry rather than a computed `user-<uid>.slice` path, so no uid or systemd layout is hard-coded and a tighter limit lower in the tree is caught too.
  The user slice is an ancestor of the dobby and of every tmux pane reed starts.
- The margin is an absolute task count, not a percentage: the incident cost about 500 tasks a run, and an absolute count stays meaningful when the operator raises `pids.max`.
- The limit is read before each start, not once per tick, since a reading is a few small file reads.
- The hold delays only new starts; passes of runs already started are never held, as holding them frees no processes.
- The hold episode lives in process only, like the dobby's other log episodes, and adds nothing to the state record.

## Not covered

- The dobby view and `lym beam status` show no reason for a task the hold keeps back; the doc names this, and the dobby log records the hold.
- A hand `lym batten run` start is not held.
- cgroup v1 and non-Linux hosts read as no limit and start runs as before.
- A first pass forks little at its start, so several starts in one tick, or by several dobbies, can each pass the check before the earlier ones grow; the margin has to absorb that.
