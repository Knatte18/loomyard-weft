# Daemons wait on events; orch reload lands on time; revival by start time

Long-lived `lyx` processes now wait on events or on second-scale, backing-off polls, and a guard pins the rule.
The orch's post-compaction reload no longer waits minutes behind a disagreeing pane probe.
Reed revives sessions in the order the runs were seeded, read from a recorded start time instead of the seed file's mtime.

Closes #481

## What changes

### The rule: `PATTERN-daemon-wakeups`

- A new PATTERN entry, with a background file and a guard test under `cmd/lyx/`.
  An unbounded wait blocks on an event source where one exists, and any periodic wake is at least one second apart.
  A wake that spawns a process backs off while nothing changes.
  A sub-second delay in such a loop is a one-shot after an event, announced by `//lyx:one-shot`.
- The guard fails on an unannounced sub-second duration in a declared file and on a `time.NewTicker` in an undeclared production file.
- A configured interval below the floor is floored to one second with one Warn, never refused.
  The floor applies to the orch watcher's `poll_interval_ms`, shuttle's `poll_interval_ms` and batten's `poll_interval_s`.
  Shuttle's template default moves from 500 ms to 1000 ms.
- New package `internal/fswatch` wraps `github.com/fsnotify/fsnotify` (a new direct dependency) as the one file-change event source, and joins `PATTERN-leaf-packages` with its closed import set.

### `lyx reed watchdog`

- The per-worktree resize loop waits on `fswatch` for its signal file and makes no stat call while no event arrives.
  If the watch cannot be opened it stays in poll mode and logs that once.
- The hub discovery loop and the name-repair loop double their cadence to a ceiling while nothing changes, and reset on a change.
  A hub-level discover signal, touched by a cold session boot, wakes the discovery loop at once.
- The report `docs/benchmarks/daemon-idle-cpu.md` records the measurement.
  `list-sessions` spawns fell from 59 to 8 per five minutes, but idle CPU did not visibly move: about 100 s per hour before and 102 s after, from one sample each.
  Nearly all the CPU is in children the measurement could not identify, so the report says the remaining cost needs its own measurement.

### Orch reload after compaction (#481)

- The resume pointer reaches the session through a provider `SessionStart` hook (matcher `compact`) right after a compaction, ahead of any next message.
  A hidden verb, `lyx orch resume-context`, renders it as the provider's additional-context JSON and writes a delivery mark.
  Shuttle's launch spec carries the hook command provider-neutrally; the event name and JSON shape stay in `claudeengine`.
- The typed pointer step stays as the fallback: with no mark, a stale mark, or a mark whose text differs from `PendingResume`, the watcher types it as before.
- Shuttle gains one readiness reading built on the hook-derived session state.
  It replaces the pane idle probe as the gate for every watcher injection and for the verified send's idle wait.
  It reads ready on a turn end, and on `asking` or `busy`/`background` with an empty input box.
  It holds, with a reason, on a running turn, an unmatched turn start, a draft in the input box, and a dead or too-short pane.
  The pane probe is the fallback only when the state is `unknown`.
- Every held step is logged once per change with its reason and recorded in `State.Stuck`.

### Revival order

- `shedrun.Seed` gains `started_at`, stamped by `WriteSeed` when absent.
  `hubgeom`'s spawn order sorts by it ascending, puts undated seeds after the dated ones by name, and no longer reads the seed's mtime.

## Design decisions

- Only the watchdog's three loops move to events or backoff.
  The orch watcher, batten's Run-Shed wait, shuttle's `Wait` and webster's poll keep their polls, which comply as floored.
- Changing shuttle's default from 500 ms to 1000 ms affects every run's wait loop; a run in flight keeps its config until the next reconcile.
- When the draft case holds the color and plugins steps, the hook verb has already written a mark naming `PendingResume`, so the watcher skips its own pointer step.
- `internal/shell` gains `ChangeDir`, so the hook command changes to the prime's directory before running the verb and still renders the prime's pointer from another worktree.

## To check by hand

- Watch the first live compaction after landing: read the transcript to confirm the hook's context landed ahead of the next message, since a dropped output with a mark written skips the pointer.
- Watch for a typed step landing in a dialog the Notification hook does not report.
- On a host with a low inotify `max_user_instances` and many worktrees, the watchdog falls back to poll mode; check it logs this once per loop.
- Findings left open for hand review:
  - `d11-hook-cwd-drifts`: the hook's working directory when the session's cwd is another worktree, in the `lyx orch resume-context` hook command.
  - `d11-pointer-names-stale-role-file`: the pointer can name a role file that is stale; the verb renders the role file first outside the `resuming` phase.
  - `card11-skip-phase-end-offset`: the reload's skip-phase end offset in the watcher's reload step.
  - `orch-resume-absent-state-no-refusal`: `lyx orch resume-context` with absent orch state should print a JSON error and write no mark.
  - `overview-opener-decision-vs-card6-tests`: the opener of `docs/overview.md` against the tests of the readiness reading.
  - `shuttleengine-doc-shadow-mode-stale`: the `shuttleengine` `doc.go` wording on shadow mode may be stale.
- `PATTERN.md` is within a few bytes of its size budget; the next entry needs room freed first.
- The benchmark ran against a sandbox hub; its after figure was taken at an earlier commit of this branch, so rerun the procedure in the report against the final build if the CPU figure matters.
