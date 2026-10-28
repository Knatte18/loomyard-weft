# Decision record — dobby-task-limit-hold

## Goal

Before dobby starts a new run, it reads the cgroup task limit its own process sits under on Linux, and while the free tasks under that limit are fewer than a configured margin it holds the start and logs both numbers.
Five loom runs and a darn run took the user slice to `pids.current 2997` of `pids.max 3000`, where every fork failed; the hold keeps dobby from adding a run to a machine that is already near that point.

## Scope

In:

- A reader of the task limit in `internal/proc` (D1, D2).
- A beam.yaml key for the margin (D3).
- The hold in dobby's start step, and its log lines (D4, D5).
- The wiring in `internal/beamcli` and the docs: `internal/proc/doc.go`, `internal/dobbyengine/doc.go` and the beam.yaml template comment (D6).

Out:

- Passes of runs already started: a continue or full pass of a run in flight is never held.
- The dobby view and `lym batten overview`: a task the hold keeps back shows no reason there.
- A hand `lym batten run` start.
- cgroup v1 hierarchies and non-Linux hosts, which read as no limit.
- EAGAIN retries and the `host-resources` transient halt class, which `transient-halt-recovery` landed.

## Decisions

### D1 — The limit is the tightest pids level of the dobby's own cgroup ancestry

The reader takes the process's cgroup v2 path from `/proc/self/cgroup` (the `0::` line) and walks from that cgroup up to the root of `/sys/fs/cgroup`.
At each level with a numeric `pids.max` it reads `pids.current`, and it reports the level with the fewest free tasks (`pids.max` minus `pids.current`) as current, max and the cgroup path.
A level whose `pids.max` reads `max`, or lacks the files, sets no limit.

Rationale: the user slice (`user-<uid>.slice`) is the level that ran out in the incident, and it is an ancestor of the dobby and of every tmux pane reed starts; walking the ancestry finds it without hard-coding the uid or slice layout, and also catches a tighter limit set lower down.

### D2 — No readable limit reads as no hold

The reader reports found false when `/proc/self/cgroup` is unreadable, holds no cgroup v2 line, or no level of the ancestry has a numeric `pids.max`; a malformed number at a level is skipped as no limit at that level.
dobby starts runs as today then.

Bound: a host whose limit sits outside the dobby's own ancestry, or on cgroup v1, is never held.

### D3 — beam.yaml `task_headroom`, default 500, 0 never holds

`task_headroom` is the fewest free tasks under the limit for dobby to start a run; the template default is 500, and 0 turns the hold off.
A negative value is refused naming the key, as for the other beam.yaml keys.

Rationale: six runs reached 3000 tasks, about 500 a run, so 500 is about one run's worth of room; beam.yaml already carries `max_runs`, the other cap on starts.

### D4 — The hold is checked before each start

In the start step, right before dobby spawns a task's first pass, it reads the limit through a new `Deps` seam; while free tasks are fewer than `task_headroom`, it starts no task in that tick.
A held task gets no refused-start record, notice or backoff, and is tried again on the next tick that reaches it.
A nil seam, `task_headroom: 0` and a not-found reading start as today.

Bound: the hold only delays a start; it never stops, pauses or fails a run, and passes of runs already started go on.
Between ticks the timer waits at most a minute, so a start resumes within about a minute of the headroom returning.

### D5 — One log line per hold episode

The first held start of an episode logs `hold starts: <current> of <max> tasks in <cgroup>, below task_headroom <margin>` to the dobby log, and the next start that the reading lets through logs `starts resume: <current> of <max> tasks in <cgroup>`.
The episode is kept in process only, like the other log episodes.

Rationale: the dobby log records changes only, and the board entry asks for both numbers.

### D6 — Wiring and docs

`internal/beamcli`'s `hubDobbyDeps` fills the seam with the `internal/proc` reader.
`internal/proc/doc.go` names the reader, and `internal/dobbyengine/doc.go` states the hold in its passes section, with the view's "starts on the dobby's next tick" sentence bounded by it.

## Constraints

- `PATTERN-cwd-resolution`: dobbyengine takes the reading through a seam and imports nothing new from the CLI side.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: the reader is tested over a temp-dir fixture of the cgroup files, and the hold through the fake `Deps` the start tests already use; no test reads the host's real cgroup.
- `PATTERN-tolerant-records`: the hold adds nothing to the state record.

## Auto-mode assumptions

- The hold is per dobby: each beam's dobby reads the same limit and holds on its own.
- A log line, with no orch notice, is enough report of a hold, as the board entry asks only for logging.

## Open risks

- A run's first pass forks little at its start, so several dobbies, or one tick starting several tasks, can each pass the check before the earlier starts grow; the margin has to absorb that.
- An operator who raises `pids.max`, as on the machine where this runs now, sees no hold at all unless they also tune the margin.

## Acceptance criteria

- The reader reports the tightest level of a fixture ancestry with current, max and path, skips `max` and missing levels, and reports not found for an absent or v1-only `/proc/self/cgroup` fixture and for an ancestry with no numeric limit (D1, D2).
- beam.yaml loads `task_headroom: 500` from the template, and a negative value is refused naming the key (D3).
- A ready task is not started while the reading's headroom is below `task_headroom`; it starts once the headroom returns; `task_headroom: 0`, a not-found reading and a nil seam start it as before; a held task leaves no start record (D4).
- The hold and the resume each write one dobby log line with both numbers, once per episode (D5).
- `hubDobbyDeps` fills the seam, and the docs name the reader and the hold (D6).
