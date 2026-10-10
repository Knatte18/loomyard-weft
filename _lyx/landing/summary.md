# Daemon wakeups: fswatch, reed backoff, hook-driven orch reload

Waits in lyx's long-lived processes now follow one rule, and the orch reload after a compaction no longer waits on a pane probe that can disagree with the session's own hook state.

Closes #481

## What changes

- **Rule and guard.**
  `PATTERN-daemon-wakeups` says an unbounded wait blocks on an event where one exists and otherwise polls at least one second apart, backing off while nothing changes when each wake spawns a process.
  `cmd/lyx/daemonwakeups_test.go` scans a declared list of wait files for sub-second literals not marked `//lyx:one-shot`, and fails on `time.NewTicker` or `time.Tick` in any unlisted production file.
  The scan sees literal shapes only; a computed sub-second period passes it and is caught by review.
- **`internal/fswatch`.**
  A new leaf package over `github.com/fsnotify/fsnotify` v1.9.0 (new direct dependency) is the one file-event source.
  When a watch cannot be opened the caller falls back to its poll cadence and logs once.
- **`lyx reed watchdog`.**
  The resize loop waits on `reed-resize.signal` through `fswatch` instead of a 100 ms stat poll.
  Discovery backs off from 5 s to 60 s while the session set is unchanged, and wakes on `reed-discover.signal`, which reed's cold session boot touches.
  Name repair backs off from 10 s to 60 s.
  Windows stays poll-only for the resize loop.
- **Orch readiness (#481).**
  `shuttleengine` gains a provider-neutral readiness reading computed from the hook-derived session state plus a draft check.
  Every watcher injection and `awaitIdleSession` go through it, so the watcher and the send it makes cannot disagree.
  The pane probe is now the fallback for an `unknown` state.
  The watcher logs `orch: injection held` with the reason and the wait so far, shows the reason in `lyx orch status`, and logs `orch: injection typed` when the step goes out.
- **Resume pointer by hook.**
  The orch run gets a `SessionStart` hook on the `compact` matcher.
  It records a `session-start` signal and runs the new hidden verb `lyx orch resume-context`, which prints the pointer as the provider's additional-context JSON.
  The verb writes a delivery mark after a successful render; the watcher skips the typed pointer step only when the mark is at or after the boundary and its text equals `PendingResume`.
  The typed pointer remains the fallback.
  `/color` and `/reload-plugins` stay typed and readiness-gated.
  The hook command is built through `internal/shell` (new `ChangeDir`) so it runs from the prime whatever the session's cwd.
  `RenderRoleFile` now writes through a temporary file and a rename.
- **Poll floors.**
  The orch `poll_interval_ms`, shuttle's `poll_interval_ms` and batten's `poll_interval_s` are floored to one second with one Warn rather than refused.
  Shuttle's template default moves from 500 ms to 1000 ms, so the liveness probe runs every 10 s instead of 5 s.
  A run in flight keeps its config until reconciled.
- **Revival order.**
  `shedrun.Seed` gains `started_at`, stamped by `WriteSeed` when absent, and `hubgeom` orders spawns by it instead of the seed file's mtime.
  Seeds written before this change are undated, so they revive in worktree-name order once.
- **Docs.**
  `doc.go` files, `docs/overview.md` and `PATTERN.md` are updated.
  `PATTERN-daemon-wakeups` sits under an existing section, and the overview was shortened (wording only) to fit its size cap.

## Design decisions to know

- The readiness reading is taken over the folded state's name and cause, not the newest signal.
  An unmatched turn start is released as the verified send releases it today.
- A session event the hook does not report, such as a menu with no box drawn, can let an injection through.
  The verified send's landed check bounds that.
- Only the watchdog's three loops were converted to events.
  The orch watcher, batten's Run-Shed wait, shuttle's `Wait` and webster's poll keep floored second-scale polls.

## To check by hand

- **Idle CPU did not drop.**
  `docs/benchmarks/daemon-idle-cpu.md` records watchdog CPU at about 100 s per hour before and 102 s after, with `list-sessions` spawns down from 59 to 8 per five minutes.
  The change cuts wakeups and spawns but not measured CPU; check the report's method and whether the goal of lower idle cost holds.
- **Live compaction.**
  Watch the first live compaction reload in the transcript to confirm the hook's context landed ahead of the next message.
  A render the provider drops after the mark is written would skip a pointer the session never got.
- **Typed step in a dialog.**
  Watch a live smoke run for a typed step landing in a dialog the Notification hook does not report.
- **Findings left open by plan review**, each addressed by a fixer but not re-reviewed:
  - the skip of the typed pointer step against the phase-end offset after the boundary;
  - `lyx orch resume-context` with absent state, where no refusal is raised;
  - the overview opener wording against the tests that pin it;
  - `internal/shuttleengine/doc.go` still describing a shadow mode that no longer exists.
- **inotify limits.**
  A host with a low `max_user_instances` falls back to poll mode, logged once per loop.
- **PATTERN.md headroom.**
  The overview is within a few bytes of its cap again after this change's additions; the next entry needs room freed first.
