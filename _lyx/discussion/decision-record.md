# Decision record: daemon-wakeups

## Goal

lyx's long-lived processes wait cheaply and wake on time:

- every unbounded wait in a `lyx` process blocks on an event where its substrate offers one and otherwise polls at a second-scale, backing-off cadence, pinned by a guard, with `lyx reed watchdog` converted first and its idle CPU measured before and after;
- the orch's resume pointer enters the session's context through a provider session-start hook right after every compaction, ahead of any next message, and the watcher types the remaining steps as soon as the session's own hook-derived state says the session is between turns, instead of waiting on a pane probe that can disagree for minutes (#481), and logs why each step waited;
- reed revives sessions in the order the runs were seeded, read from a start time recorded in the seed, not from the seed file's mtime.

## Scope

In:

- A new PATTERN entry, `PATTERN-daemon-wakeups`, with a background file, and its guard test under `cmd/lyx/`.
- A new package `internal/fswatch` wrapping `github.com/fsnotify/fsnotify` (a new direct dependency, already in the module cache) as the one file-change event source for every lyx process.
- `lyx reed watchdog`: the per-worktree resize loop waits on `fswatch` for its signal file; the hub discovery loop and the name-repair loop back off while nothing changes and wake on a hub-level discover signal; a before-and-after idle CPU measurement recorded under `docs/benchmarks/`.
- The orch's resume pointer after a compaction delivered through the provider's session-start hook, recorded in the events file, with the typed pointer step kept as the fallback; a new `lyx orch` verb renders the hook's context; shuttle's launch spec carries the hook command provider-neutrally.
- The orch watcher: one readiness reading from shuttle, built on the hook-derived session state, replaces the pane idle probe as the gate for every injection; the verified send's idle wait uses the same reading; every held step is logged with its reason.
- Poll floors: the orch watcher's `poll_interval_ms`, shuttle's `poll_interval_ms` and batten's `poll_interval_s` are floored at one second, and shuttle's template default moves from 500 ms to 1000 ms.
- `shedrun.Seed` gains `started_at`; `hubgeom`'s spawn order reads it.
- Docs in the same commit: `doc.go` of `fswatch`, `reedengine`, `reedcli`, `orchengine`, `shuttleengine`, `shedrun` and `hubgeom` where their behaviour changes; `docs/overview.md`'s module table for the new package; `PATTERN.md`.

Out:

- Converting any loop other than the reed watchdog's three loops to events.
  The orch watcher, batten's Run-Shed wait, shuttle's `Wait` and webster's poll keep their second-scale polls, which comply with the rule as floored here.
- Creel, `lyx-halt`, `idle-notice-after-sleep`, `spawn-audit-log`: the rule is written so their daemons fall under it, but nothing of them is built.
- tmux hooks for session creation and closure, and a tmux control-mode client.
- Any change to the pane idle probe's recognition of Claude Code's screen.
- Typing the resume steps into Claude Code's input box while a turn runs, in the hope they queue ahead of the operator's next message; the hook route in D11 is how the pointer gets ahead of it.
- Delivering the skills load or the pointer after `/clear` through the hook; the clear cycle keeps its typed sequence.

## Decisions

### D1. The rule: `PATTERN-daemon-wakeups`

Decision.
A new PATTERN entry in a new section "Daemons and waits", one line, with a background file `pattern/PATTERN-daemon-wakeups.md`:

> `PATTERN-daemon-wakeups` — Waiting without a bound of its own in a `lyx` process (a daemon loop, or a wait on an agent, a child run or a lock that may last hours): it blocks on an event source where one exists, file changes through `internal/fswatch` and nothing else, and any periodic wake is at least one second apart and backs off while nothing changes when each wake spawns a process; a sub-second delay in such a loop is a one-shot after an event, announced by `//lyx:one-shot`. (test)

The background file states:

- what counts as an unbounded wait: the detached daemons (`lyx reed watchdog`, `lyx orch watch`), batten's Run-Shed wait, shuttle's `Wait`, webster's long poll, `lyx shed status --watch`;
- what does not: a bounded wait that ends within a minute (a startup handshake, a send's settle, a lock retry with a short bound), which stays free to use sub-second delays;
- the floor is one second for any periodic wake, and a wake that spawns a process (tmux, git, `lyx`) or captures a pane backs off from its base cadence by doubling to a ceiling the loop names while consecutive wakes find nothing changed, and resets to the base on a change or on an event;
- a configured interval below the floor is floored to it with one Warn, never refused, since a config value must not stop a daemon;
- the event source for files is `internal/fswatch`; a tmux hook that touches a file is the way tmux state becomes a file event.

Rationale.
The board asks for a rule for all daemons, not a fix for one, with a guard so a new daemon cannot ship with a sub-second poll unannounced.
Scoping the rule to unbounded waits keeps it about the cost of hours of nothing happening, which is the measured problem, and leaves bounded waits alone, where sub-second delays are correct.

### D2. The guard: a declared-file scan in `cmd/lyx/daemonwakeups_test.go`

Decision.
One untagged test, marked `//lyx:guard`, in the manner of `cmd/lyx/tierpurity_test.go`:

- It holds the declared list of unbounded-wait files: `internal/reedengine/watchloop.go`, `internal/reedengine/namerepair.go`, `internal/reedcli/watchdog.go`, `internal/orchengine/watchloop.go`, `internal/battenshed/innerrun.go`, `internal/battenshed/primelockwait.go`, `internal/shuttleengine/wait.go`, `internal/websterengine/poll.go`, `internal/shedverbs/status.go`.
- In every declared file it fails on a sub-second duration literal (`N * time.Millisecond` with N below 1000, any `time.Microsecond` or `time.Nanosecond` multiple) and on an integer constant whose name ends in `MS` or `Ms` with a value below 1000, unless the line carries `//lyx:one-shot <reason>`.
- It fails on any production `.go` file outside the declared list that calls `time.NewTicker` or `time.Tick`, naming the file, so a new loop is listed or explained in the same commit.
- Each declared loop's package has a unit test asserting its production timing's periodic fields are at or above one second and that a configured interval below the floor is floored: `watchDefaultTiming`, `watchdogDefaultTiming`, the orch `Config.PollInterval`, shuttle's `pollInterval`, batten's recipe `poll_interval_s` floor.

Bound: the scan sees literal shapes only.
A computed sub-second period, or a `time.Sleep` loop in an unlisted file, passes the scan and is caught by review, which the background file says.
`//lyx:one-shot` lets a sub-second literal through on any line that carries it, so the guard enforces announcement, not absence, which is what the board asks.

Rationale.
A literal scan over a declared set is the repo's existing guard idiom, cheap, tier 1, and it closes the dodge of not listing a new loop by catching tickers anywhere.
Behavioural floor tests catch the computed cases the scan cannot.

### D3. `internal/fswatch`: one file-event package over fsnotify

Decision.
A new leaf package, added to `PATTERN-leaf-packages`, importing only the standard library, `github.com/fsnotify/fsnotify` and `internal/logger`.
Its surface is one watcher over one directory: `Watch(dir string, names ...string) (*Watcher, error)` delivering an event per create, write, rename or remove of a named entry (or any entry when no names are given) on a channel, with `Close`.
It coalesces nothing and debounces nothing; the caller owns timing.
An `fsnotify` failure at open or add (the inotify watch limit, an unsupported filesystem) returns an error, and the caller falls back to its poll cadence and logs that once.
Tests are untagged: they create and touch files in `t.TempDir()` and need no spawn.

Rationale.
The board asks for inotify and its Windows and macOS equivalents through one shared package; fsnotify is that, in the module cache already, and a thin wrapper keeps provider details out of every daemon.
A leaf with a closed import set is the repo's way to keep a utility package from growing feature edges.

### D4. The resize loop waits on the signal file through `fswatch`

Decision.
In `internal/reedengine/watchloop.go`, signal mode blocks on a select over the `fswatch` events for `reed-resize.signal` in the worktree's state dir, the one-shot debounce timer (200 ms, `//lyx:one-shot`), the one-shot retry timer after a failed apply (`//lyx:one-shot`), and `ctx.Done()`.
`watchdogSignalTick` and the 100 ms stat poll go.
An event is handled as a stat tick is today: stat, remove, `Signal(now)`; the quiet window arms the debounce timer; its firing runs the apply.
Poll mode and dormant mode keep their 2 s and 60 s tickers.
Where `fswatch.Watch` fails, the loop stays in poll mode, logs that once at Warn, and never promotes to signal mode.
Windows stays poll-only as today, since its hook cannot touch the file.

Rationale.
The signal file already exists and the hook already writes it; only the wait changes, from ten stats per second to zero while nothing resizes.

### D5. Discovery and name repair back off and wake on a discover signal

Decision.
`runWatchdogLoop` in `internal/reedcli/watchdog.go` runs its `list-sessions` cycle at a cadence that starts at 5 s, doubles after every cycle whose session set equals the previous cycle's, and caps at 60 s; a cycle whose set differs resets it to 5 s.
It also wakes at once on `reed-discover.signal` in `fabricengine.HubScratchDir(hub)` through `fswatch`, consumes it, runs a cycle and resets the cadence.
The signal is touched by `lyx reed watchdog` itself when it finds the lock already held, before it exits: every `up`, `resume` and `attach` already spawns that child, so a session booted by lyx is discovered within a cycle of its boot with no new edge in the verbs.
Idle exit and orphan reap keep their counts of three consecutive cycles; at the backed-off cadence that is up to three minutes, which the reap's own comment already prefers over haste.
`WatchNames` runs its pass at 10 s, doubling to 60 s while a pass repaired nothing, reset to 10 s by a pass that repaired something.
Both loops' timing lives in their existing timing structs so tests drive them fast.

Rationale.
The daemon's idle cost is dominated by process spawns, two tmux calls per worktree every 10 s plus one per hub every 5 s, not by the stat tick alone; the rule's backoff clause exists for exactly this.
A discover signal from the spawn attempt lyx already makes is a lyx-owned event that needs no tmux server hooks and works on Windows.

### D6. Idle CPU is measured before and after, and the report lands under `docs/benchmarks/`

Decision.
The measurement is a shell procedure, no Python: start a hub with at least three idle reed sessions, find the daemon's pid through the lock holder or `pgrep -f "lyx reed watchdog"`, read `utime+stime` from `/proc/<pid>/stat` at the start and after ten minutes, and report CPU seconds per hour.
The after figure also counts tmux spawns per minute, read from the hub's durable log at Debug.
The report is `docs/benchmarks/daemon-idle-cpu.md`, a measurement report of one run on one date, with the before figure taken on `main` before the change.

Rationale.
The board asks for before and after on the watchdog; a measurement report is the one document kind that may pin numbers, and `docs/benchmarks/` is where the repo keeps them.

### D7. One readiness reading, from hooks, gates every orch injection and every verified send

Decision.
`shuttleengine` gains a provider-neutral readiness reading over a run, computed from the hook-derived session state (`SessionState`) plus one pane check for a draft:

- Ready: the newest hook signal is a turn end, so the state is `idle-done`, `idle-stalled` or `asking` with cause `awaiting-input`, or `busy` with cause `background`; no turn start is unmatched; and the input box, where the pane shows one, is blank.
- Held, with a reason string: a turn is running (`busy`/`turn`, or an unmatched turn start); a dialog is open (`asking` with cause `ask`); a draft is in the input box, read through the provider's `InputBoxText`; the pane is too short to show a box.
- The pane's running-turn needle and a pane that shows no box no longer hold a ready reading; they are logged as a disagreement, with the pane's last lines at Debug.
- An unreadable session state falls back to today's pane probe and logs that at Debug.

`orchengine.Session.SessionIdle` returns this reading in place of `IdleProbe`, keeping `TooShort`, and `Watcher.probeIdle` stays the one door every watcher probe goes through, so the idle phase's triggers, notice delivery, the handoff gate, clearing, the compaction boundary's completion and every reload step gate on it.
`awaitIdleSession` in `internal/shuttleengine/send.go` uses the same reading in `busyReading`, so the watcher and the send it then makes cannot disagree.

Bound: this lets an injection through while Claude Code is in a screen state the hooks do not report and the box is not drawn, such as a menu; it is bounded by the verified send's landed check, which retries or reports a submission that did not land, and by the fact that only the watcher's own commands and prompts are typed, never operator text.
A draft still holds the typed steps, so `/color` and `/reload-plugins` after a compaction wait while the operator is typing, and the log says so; the resume pointer does not wait, since D11 delivers it through the session-start hook ahead of the operator's message.

Rationale.
The 2026-10-10 log shows the state `asking` beside a pane reading not idle at every turn end for three minutes, with no reason recorded; `shuttle-state`'s staged plan names the orch watcher as the next consumer to move from the pane to the hook-derived state, and this is that move.
Keeping the draft check keeps the one hazard the pane alone can see.

### D8. The watcher logs why each step waited

Decision.
Whenever a tick holds an injection on a not-ready reading, the watcher logs `orch: injection held` at Info with the phase, the reload step, the reason and the time since the phase or step began, once per change of reason in the manner of `markStuck`, and records the reason in `State.Stuck` so `lyx orch status` shows it.
When the step is typed, it logs `orch: injection typed` with the same fields and the wait's length.
The compaction boundary's time is logged when found, so the delay from boundary to reload is readable from `watch.log` alone.

Rationale.
The board's third expectation; on 2026-10-10 the log held four disagreement lines and no reason.

### D9. Poll floors: one second, floored not refused

Decision.
`orchengine.Config.PollInterval`, `shuttleengine`'s `pollInterval` and batten's `poll_interval_s` floor a value below one second to one second and log one Warn naming the key and the floor; shuttle's `template.yaml` default becomes `poll_interval_ms: 1000`, and `liveness_every_n_polls` stays, so the liveness probe runs every 10 s instead of 5 s.
The existing non-positive floors are replaced by the one-second floor.

Bound: a wait loop can no longer be configured faster than one second, so a test that wants a fast loop injects a clock or a timing struct, never a config value.

Rationale.
The guard's floor tests need a floor to assert; a refusal would stop a daemon over one config key, which `PATTERN-config-strictness`'s degrading side avoids.

### D10. The seed carries `started_at`, and spawn order reads it

Decision.
`shedrun.Seed` gains `StartedAt string` with tag `json:"started_at,omitempty"`, RFC3339 UTC.
`shedrun.WriteSeed` stamps it with the current time when the seed it is given has none, so every seeding site gets it without a change; a caller that passes one keeps it.
`hubgeom.runStartTime` reads the seed through `shedrun.ReadSeed` and parses `started_at`; a seed without it, or one that does not parse, is undated, and the mtime is never read.
Undated pairs sort after dated ones by worktree name, as today.

Bound: runs seeded before this lands are undated once, so their revival order after the next server restart is by name; nothing else depends on the order.

Rationale.
A time inside the file survives a checkout or re-materialization, which is what moves the mtime; `shedrun` is the sole Seed codec, so the stamp belongs in its writer.

### D11. The resume pointer rides the provider's session-start hook after a compaction

Decision.
The orch run's provider settings gain a session-start hook on the provider's compaction matcher (Claude Code: `SessionStart` with matcher `compact`, which fires after a manual and an automatic compaction alike) with two commands:

- the recording command every other hook already uses, appending a stamp line and the payload to the run's events file, so shuttle parses a new `session-start` signal carrying the source;
- `lyx orch resume-context`, a new hidden verb run from the orch's own cwd, which renders the pointer the watcher would type, the resume prompt naming `State.LastHandoff` when the persisted phase is `compacting` and the reload prompt naming the role alone otherwise, and prints it as the provider's additional-context JSON and nothing else, so the pointer is in the session's context before any next message is taken.

Shuttle's launch spec carries this as one provider-neutral field, a context-after-compaction command, set by `orchcli` alone; `claudeengine` renders it into the hook with the matcher, and every other run leaves the field empty and gets no such hook.
The watcher's reload after a compaction, requested or automatic, keeps the color and plugins steps on the typed, readiness-gated path, and skips its pointer step when a `session-start` event with source compaction was read after the boundary; when none has been read by the time the plugins step is done, it types the pointer as today, the fallback for a hook that did not fire.
The phase ends at the first turn end after the boundary as today, whether the operator's message or the typed pointer started that turn.
The watcher logs the hook's delivery with its time when it reads the event (D8).

Bound: this adds a hook edge, context injected into the orch session at every compaction; it is bounded to the orch run, the one spec that sets the field, and to the rendered pointer text, which asks the session to read the role and the note and executes nothing.
A verb failure prints its JSON error, which Claude Code shows and does not add as context, and the typed fallback delivers the pointer.

Rationale.
The board's first expectation says the resume steps follow the boundary before the operator's next message is taken or are queued ahead of it; with a draft in the box the typed path cannot do either without typing over the operator's text, and a hook whose output enters the context right after the compaction does both without touching the pane.
Keeping the typed pointer as the fallback keeps today's behaviour when the hook does not fire.

## Constraints

- `PATTERN-leaf-packages`: `internal/fswatch` is added to the entry with its closed import set.
- `PATTERN-shuttle-provider-seam`: the readiness reading lives in `shuttleengine` over the existing provider seams (`IdleSession`, `InputBoxText`, `ParseSessionSignals`); no Claude specifics leave `claudeengine`.
  The session-start hook's event name, matcher and JSON shape live in `claudeengine`; the spec field and the `session-start` signal kind are provider-neutral.
- `PATTERN-cli-cobra`: `lyx orch resume-context` is a hidden verb under the orch subtree with a `Short` and JSON errors; its success output is the provider's hook JSON, which is the one shape the hook accepts.
- `PATTERN-stencil-ownership`: the verb renders from the told stencils dir through the same render functions the watcher uses.
- `PATTERN-orch-pane-single-writer`: unchanged; the hook writes context, not the pane, and the watcher stays the only writer into the orch pane.
- `PATTERN-shed-run-directory`: only `shedrun` declares the Seed codec; `hubgeom` reads through it.
- `PATTERN-told-geometry`: `reedengine` is told its state dir and derives nothing; `fswatch` takes a told directory.
- `PATTERN-durable-vs-ephemeral-state`: the discover signal lives under the hub scratch dir, the resize signal under the worktree's `.lyx` as today.
- `PATTERN-spawn-observability`: the discovery cycle's `list-sessions` and the name-repair pass stay logged at Debug; the measurement counts those lines.
- `PATTERN-test-speed`: every new timing is a struct field or parameter; `fswatch` tests are untagged; the watchdog's event path is tested with a fake event channel in tier 1 and once live in the existing `tmux` tier.
- `PATTERN-test-economy`: the readiness reading extends the existing session-state tests' tables where a scenario fits.
- `PATTERN-comment-line-breaks` and semantic line breaks in every `.md` file.
- Test budget: `cmd/lyx` gains one test, `internal/fswatch` needs a new row, and `shuttleengine`, `claudeengine`, `orchengine`, `orchcli`, `reedengine`, `reedcli`, `shedrun` and `hubgeom` may need a raise; the card that adds them lists `cmd/lyx/testdata/test-budget.yaml` under `Edit:`.
- No Python anywhere, including the measurement procedure.

## Auto-mode assumptions

- The 2026-10-10 pane reading's cause is unknown; the prime's `watch.log` records the disagreement and no reason.
  The design holds under either cause: a probe misread no longer blocks, and a draft holds with its reason logged.
- The daemon's idle CPU is dominated by process spawns rather than by the stat tick; D5 is justified either way by the rule's backoff clause, and D6's measurement settles it.
- Claude Code accepts a typed prompt while background tasks run, so `busy`/`background` reads ready.
- Changing shuttle's wait default from 500 ms to 1000 ms is acceptable to the operator; it doubles no visible latency beyond one poll.
- The discover signal from the lock-held child is enough event coverage; a session created by hand outside lyx verbs is found within one backed-off cycle.
- `fsnotify` v1.9.0 is the version to pin.
- Claude Code's `SessionStart` hook with matcher `compact` fires after manual and automatic compaction, runs in the session's cwd, and its additional-context output precedes the next user message in the context; the typed pointer stays as the fallback in case any of this differs.

## Open risks

- The readiness reading lets a ready verdict through when the pane shows no input box while hooks say the turn ended; if Claude Code draws a dialog the Notification hook does not report, text could land in it.
  Mitigated by the verified send's landed check; the first live smoke run of a compaction reload should be watched.
- The shuttle poll default change touches every run's wait loop; a run in flight keeps its config until reconciled, so the change lands at the next binary-change reconcile.
- inotify watch limits: one watch per worktree state dir plus one per hub is small, but a hub with many worktrees on a low `max_user_instances` host falls back to poll mode, which is logged once per loop.
- Backed-off orphan reap takes up to three minutes; a deliberately removed worktree keeps its session that long.
- The hook route rests on Claude Code's session-start hook semantics as assumed above; the first live compaction after landing should be read in the transcript to confirm the context landed ahead of the next message.
  If it did not, the typed fallback delivers the pointer and the draft case stays as late as today.
- `/color` and `/reload-plugins` still wait on a draft; the session answers the operator's message with the right role and note but with stale plugins until the next ready tick.

## Acceptance criteria

- The guard fails on a sub-second duration literal without `//lyx:one-shot` in a declared file, on a `time.NewTicker` call in an undeclared production file, and passes on `main` after this task.
- Each declared loop's package test proves the production timing's periodic fields are at or above one second and a configured value below the floor is floored with one Warn.
- `fswatch`: an event arrives for a create, a write and a remove of a named entry; unnamed entries are filtered; `Close` ends the channel; an unwatchable directory returns an error.
- The resize loop in signal mode makes no stat call while no event arrives, applies once after a burst of events settles, retries on a failed apply with the escalating one-shot delay, and abandons after the cap; the fallback to poll mode when the watch cannot be opened is logged once and never promotes.
- The discovery loop's cadence doubles to the ceiling across unchanged cycles, resets on a changed set and on a discover signal, and a discover signal runs a cycle at once; idle exit and orphan reap still need three consecutive cycles.
- A `lyx reed watchdog` that finds its lock held touches the discover signal and exits 0.
- The name-repair loop backs off while nothing drifted and resets after a repair.
- The readiness reading: ready on `idle-done`, `idle-stalled`, `asking`/`awaiting-input` and `busy`/`background` with a blank or absent box; held with a reason on `busy`/`turn`, an unmatched turn start, `asking`/`ask`, a draft in the box, and a pane too short; a pane running-turn needle beside a hook turn end reads ready and logs the disagreement; an unreadable state falls back to the pane probe.
- The orch watcher completes a compact cycle on the boundary and types the reload steps on the next tick whose readiness is ready, with the pane probe reading not idle; the same for an auto-compaction boundary read at a turn end.
- The draft case: with a draft in the input box at the boundary, the resume pointer is in the session's context before the operator's message is taken, proven by the `session-start` event in the events file preceding the operator's turn start, the color and plugins steps are held with reason `draft` and typed at the next ready tick, and the watcher skips its pointer step.
- The hook fallback: with no `session-start` event read after the boundary, the watcher types the pointer after the plugins step as today.
- `lyx orch resume-context` prints the resume prompt naming the last handoff while the phase is `compacting`, the reload prompt otherwise, as the provider's additional-context JSON; outside the prime, or with no orch state, it prints a JSON error.
- `claudeengine` renders the spec's context-after-compaction command into a `SessionStart` hook with matcher `compact` beside the recording command, and renders no such hook when the field is empty; a `session-start` payload parses to a signal carrying its source.
- Every held injection logs its reason once per change and records it in `State.Stuck`; a typed step logs its wait; the compaction boundary's time is logged.
- The verified send's `busyReading` holds on a draft and on an unmatched turn start and passes on a hook turn end beside a pane needle.
- `WriteSeed` stamps `started_at` when absent and keeps a given one; `ReadSeed` returns it; a seed written before the field decodes.
- Spawn order sorts dated pairs by `started_at` ascending, undated after them by name, and never reads the seed's mtime; a rewritten seed with the same `started_at` keeps its place.
- The idle CPU report exists under `docs/benchmarks/` with a before figure from `main` and an after figure, both from the stated procedure.
- `go test ./...` and `go test -tags integration ./...` are green, and `go vet -tags tmux ./...` and `go vet -tags llm ./...` compile.

## Notes for the plan writer

- `watchState` in `internal/reedengine/watchloop.go` is already pure and told the time; the event path can call `Signal` and `Plan` unchanged, and the select replaces the ticker branch.
- `handleWatchOutcome` need not change; the promotion to signal mode is where the `fswatch` watcher is opened, and the fallback on error stays in poll mode.
- The discover signal's file name and the lock file's name sit together in `internal/reedcli/watchdog.go`; the lock-held exit is the `!acquired` branch of `watchdogCmd`.
- The readiness reading can be built from `Runner.SessionState`, `unmatchedTurnStart` and the provider's `InputBoxText`, which already exist; `IdleProbe` can grow a `Reason` field rather than be replaced, which keeps the `Session` seam's fakes in `orchengine` tests small.
- `orchengine`'s unit tests drive `Watcher` against a fake `Session`; the #481 scenario is one more table row: compaction boundary found, fake probe not idle, fake state `asking`/`awaiting-input`, expect the reload typed on that tick; the draft case is a second row with a `session-start` event in the fake's events and a draft reason, expecting color and plugins held and the pointer step skipped.
- `settingsHooks` in `internal/shuttleengine/claudeengine/settings.go` lists the recording hooks; `SessionStart` is one more entry, with the matcher on the orch's spec field only, and `recordingCommand` already builds the stamp-and-payload command.
- `startCompacting` sets `State.LastHandoff` before `/compact` is typed, so `lyx orch resume-context` can read the note path from the persisted state while the phase is `compacting`; `RenderResumePrompt` and `RenderReloadPrompt` are the two render functions it reuses.
- `ReloadSkipsSkills` already marks a reload after a compaction; a sibling mark for the pointer delivered by hook keeps `reloadStep`'s normalisation the one place that decides which step is next.
- `hubgeom`'s `sortSpawnOrder` is pure and already tested; only `runStartTime` changes.
- The measurement procedure can live as a short section in the benchmark report itself, so the next run repeats it.
