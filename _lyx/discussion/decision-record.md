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
A new leaf package, added to `PATTERN-leaf-packages`, importing only the standard library, `github.com/fsnotify/fsnotify` pinned at v1.9.0, and `internal/logger`.
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

Bound: the event wake is a new wake source into the daemon, and any process of the operator can touch the worktree's `.lyx`; the writer lyx owns is reed's resize hook, as today.
An event costs one stat, and a remove when the file is there, and spawns no process; an event that finds no file, such as the one the loop's own remove raises, runs nothing.
An apply, the step that spawns tmux, runs only when the debounce timer fires after a quiet window, and a failed one retries on the escalating one-shot delay up to its cap, so a toucher in a loop holds the apply off rather than repeating it.

Rationale.
The signal file already exists and the hook already writes it; only the wait changes, from ten stats per second to zero while nothing resizes.

### D5. Discovery and name repair back off and wake on a discover signal

Decision.
`runWatchdogLoop` in `internal/reedcli/watchdog.go` runs its `list-sessions` cycle at a cadence that starts at 5 s, doubles after every cycle whose session set equals the previous cycle's, and caps at 60 s; a cycle whose set differs resets it to 5 s.
It also wakes on `reed-discover.signal` in `fabricengine.HubScratchDir(hub)` through `fswatch`: a wake stats the file, and only a file found there is consumed (removed), runs a cycle and resets the cadence, so the loop's own removal, or any event that finds no file, runs nothing.
The signal is touched by `reedengine`'s cold session boot, right after the session exists, at a path the engine is told on its geometry.
Every lyx path that boots a session goes through that boot, whether or not it also spawns the watchdog, so a session booted by lyx is discovered within one base cadence of its boot; an attach to a live session boots nothing and touches nothing.
Idle exit and orphan reap keep their counts of three consecutive cycles; at the backed-off cadence that is up to three minutes, which the reap's own comment already prefers over haste.
`WatchNames` runs its pass at 10 s, doubling to 60 s while a pass repaired nothing, reset to 10 s by a pass that repaired something.
Both loops' timing lives in their existing timing structs so tests drive them fast.

Bound: the discover signal is a new routing edge into the daemon, and any process of the operator can touch the hub scratch dir.
A signal wake never runs cycles faster than the 5 s base: a signal found within 5 s of the previous cycle is consumed, and its cycle runs when 5 s have passed since that cycle.
So the worst case, a toucher in a loop, costs one `list-sessions` per 5 s per hub, which is today's cadence; lyx's own writer touches once per booted session.
A session created outside lyx (by hand in tmux) touches nothing and is found within one backed-off cycle, at most 60 s.

Rationale.
The daemon's idle cost is dominated by process spawns, two tmux calls per worktree every 10 s plus one per hub every 5 s, not by the stat tick alone; the rule's backoff clause exists for exactly this.
A discover signal touched by the one boot path every lyx-booted session takes is a lyx-owned event that needs no tmux server hooks and works on Windows.

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

The reading is taken over the folded state's name and cause, never over the newest signal, and it covers every state the fold yields:

- Ready: `idle-done`, `idle-stalled` of any cause, `asking` with cause `awaiting-input`, or `busy` with cause `background`; and no turn start is unmatched; and the input box, where the pane shows one, is blank.
- Held, with a reason string: a turn is running (`busy`/`turn`, or an unmatched turn start); a dialog is open (`asking` with cause `ask`); the session is `dead`; a draft is in the input box, read through the provider's `InputBoxText`; the pane is too short to show a box.
- `unknown`, of any cause (no signal yet, no state, unproven liveness, an unreadable events file, a contradiction, a session end that leaves the process), falls back to today's pane probe and logs the cause at Debug.
- A signal kind the fold does not move on, such as D11's `session-start`, leaves the reading as it was.
- The pane's running-turn needle and a pane that shows no box no longer hold a ready reading; they are logged as a disagreement, with the pane's last lines at Debug.

An unmatched turn start is released as the verified send releases it today, through `turnStartHold`: by a later turn end, by the provider reporting that turn interrupted, or by the pane probe reading idle on every poll for `turnStartIdleOverride` (10 s).
The watcher keeps one hold across its ticks, as the send keeps one across its polls.

`orchengine.Session.SessionIdle` keeps returning an `IdleProbe`, which gains a `Reason` field and keeps `TooShort`, and `Watcher.probeIdle` stays the one door every watcher probe goes through, so the idle phase's triggers, notice delivery, the handoff gate, clearing, the compaction boundary's completion and every reload step gate on it.
`awaitIdleSession` in `internal/shuttleengine/send.go` uses the same reading in `busyReading`, so the watcher and the send it then makes cannot disagree.

Bound: this lets an injection through while Claude Code is in a screen state the hooks do not report and the box is not drawn, such as a menu; it is bounded by the verified send's landed check, which retries or reports a submission that did not land, and by the fact that only the watcher's own commands and prompts are typed, never operator text.
The idle override lets an unmatched turn start through when the pane alone says idle; it is bounded by needing that reading on every poll for 10 s, a busy poll restarting the count, and is logged at Warn as the send's release is.
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

- the recording command every other hook already uses, appending a stamp line and the payload to the run's events file, so shuttle parses a new `session-start` signal carrying the source, which the watcher logs and nothing decides on;
- `lyx orch resume-context`, a new hidden verb, which renders a pointer from the persisted state and prints it as the provider's additional-context JSON and nothing else, so the pointer is in the session's context before any next message is taken.
  The orch session moves its cwd into other worktrees, and a hook runs in the session's current cwd, so the command does not rely on it: `orchcli` builds it through `internal/shell` as a change of directory to the prime's absolute path chained with the verb, and the verb then resolves the prime through `lyxcwd` as every orch verb does.
  The command names `lyx` bare, which resolves to the binary that spawned the orch session: the hook inherits the pane's environment, whose `panebin.go` prelude puts that binary's directory first on `PATH` and exports it as `LYX_BIN`, so a session started from `.dev-bin` runs the dev binary's verb.

The verb's pointer is chosen over the persisted state:

- phase `resuming` with a `PendingResume` set: that text verbatim, since the watcher has already rendered it, which covers a watcher tick that reads the boundary and enters `resuming` before the hook runs;
- phase `compacting`: the resume prompt naming `State.LastHandoff`;
- any other phase (`idle`, `handoff-requested`, `clearing`): the reload prompt naming the role alone.
  In `idle` that is the pointer the watcher's auto-reload would type.
  In `handoff-requested` and `clearing` the watcher types no pointer for this compaction, so the hook's role pointer is the session's only reload of its role after the compaction; the cycle in flight then proceeds as today.

In every phase but `resuming`, the verb renders the role file through `RenderRoleFile` before it renders the pointer, so the file the pointer names is current when the session reads it, even when a draft holds the watcher's own reload, which renders it only once the gate reads ready.
In `resuming` the watcher has rendered it already.
Since the verb and the watcher may then render the role file at the same moment, `RenderRoleFile` writes through a temporary file and a rename, so a reader never sees a partial file.

After a successful render, and before printing, the verb writes a delivery mark in the orch dir, beside the cycle request, holding the time and the pointer text; a failed render writes no mark.

Shuttle's launch spec carries this as one provider-neutral field, a context-after-compaction command, set by `orchcli` alone; `claudeengine` renders it into the hook with the matcher, and every other run leaves the field empty and gets no such hook.
The watcher's reload after a compaction, requested or automatic, keeps the color and plugins steps on the typed, readiness-gated path.
At the pointer step it reads the delivery mark: a mark dated at or after the boundary whose text equals `PendingResume` skips the step; no mark, an older mark, or a mark with other text (the hook rendered the reload prompt while the watcher holds the resume prompt) types the pointer as today.
The watcher removes the mark when the reload ends, so a mark never carries into the next compaction.
The phase ends at the first turn end after the boundary as today, whether the operator's message or the typed pointer started that turn.
The watcher logs the hook's `session-start` event and the mark it read, each with its time (D8).

Bound: this adds a hook edge, context injected into the orch session at every compaction; it is bounded to the orch run, the one spec that sets the field, and to the rendered pointer text, which asks the session to read the role and the note and executes nothing.
A verb failure prints its JSON error, which Claude Code shows and does not add as context, writes no mark, and the typed fallback delivers the pointer.
A mark the watcher has not read by the pointer step, or one whose text differs, makes the pointer arrive twice, once by hook and once typed, which costs the session one redundant read of the role and the note and loses nothing.
A mark written for a render whose output the provider then drops would skip a pointer the session never got; that rests on the hook semantics in Auto-mode assumptions.

Rationale.
The board's first expectation says the resume steps follow the boundary before the operator's next message is taken or are queued ahead of it; with a draft in the box the typed path cannot do either without typing over the operator's text, and a hook whose output enters the context right after the compaction does both without touching the pane.
Keeping the typed pointer as the fallback keeps today's behaviour when the hook does not fire or its render fails.
Keying the skip on a mark the verb writes only after a successful render, not on the recording command's event, which is written whatever the verb does, is what makes the fallback reachable.

## Constraints

- `PATTERN-leaf-packages`: `internal/fswatch` is added to the entry with its closed import set.
- `PATTERN-shuttle-provider-seam`: the readiness reading lives in `shuttleengine` over the existing provider seams (`IdleSession`, `InputBoxText`, `ParseSessionSignals`); no Claude specifics leave `claudeengine`.
  The session-start hook's event name, matcher and JSON shape live in `claudeengine`; the spec field and the `session-start` signal kind are provider-neutral.
- `PATTERN-cli-cobra`: `lyx orch resume-context` is a hidden verb under the orch subtree with a `Short` and JSON errors; its success output is the provider's hook JSON, which is the one shape the hook accepts.
- `PATTERN-stencil-ownership`: the verb renders from the told stencils dir through the same render functions the watcher uses.
- `PATTERN-shell-mechanics-seam`: the hook command's change of directory and chaining are built through `internal/shell`.
- `PATTERN-orch-pane-single-writer`: unchanged; the hook writes context, not the pane, and the watcher stays the only writer into the orch pane.
- `PATTERN-shed-run-directory`: only `shedrun` declares the Seed codec; `hubgeom` reads through it.
- `PATTERN-told-geometry`: `reedengine` is told its state dir and the discover signal's path and derives neither; `fswatch` takes a told directory.
- `PATTERN-durable-vs-ephemeral-state`: the discover signal lives under the hub scratch dir, the resize signal under the worktree's `.lyx` as today, and the delivery mark in the orch dir under `.lyx`, beside the cycle request.
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
- Claude Code's `SessionStart` hook with matcher `compact` fires after manual and automatic compaction, inherits the session's environment, and its additional-context output precedes the next user message in the context; the typed pointer covers a hook that does not fire or whose render fails, not a rendered output the provider drops.

## Open risks

- The first live smoke run of a compaction reload should be watched for a typed step landing in a dialog the Notification hook does not report (D7's Bound).
- The shuttle poll default change touches every run's wait loop; a run in flight keeps its config until reconciled, so the change lands at the next binary-change reconcile.
- inotify watch limits: one watch per worktree state dir plus one per hub is small, but a hub with many worktrees on a low `max_user_instances` host falls back to poll mode, which is logged once per loop.
- The first live compaction after landing should be read in the transcript to confirm the hook's context landed ahead of the next message, since a dropped output with a mark written skips the pointer (D11's Bound).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d11-hook-cwd-drifts (class design, severity MEDIUM, open in rounds 2)
  - d11-pointer-names-stale-role-file (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- The guard fails on a sub-second duration literal without `//lyx:one-shot` in a declared file, on a `time.NewTicker` call in an undeclared production file, and passes on `main` after this task.
- Each declared loop's package test proves the production timing's periodic fields are at or above one second and a configured value below the floor is floored with one Warn.
- `fswatch`: an event arrives for a create, a write and a remove of a named entry; unnamed entries are filtered; `Close` ends the channel; an unwatchable directory returns an error.
- The resize loop in signal mode makes no stat call while no event arrives, applies once after a burst of events settles, retries on a failed apply with the escalating one-shot delay, and abandons after the cap; an event that finds no signal file runs nothing; the fallback to poll mode when the watch cannot be opened is logged once and never promotes.
- The discovery loop's cadence doubles to the ceiling across unchanged cycles, resets on a changed set and on a discover signal, and a discover signal runs a cycle at once when the base cadence has passed since the previous cycle; idle exit and orphan reap still need three consecutive cycles.
- A cold session boot in `reedengine` touches the discover signal; an attach to a live session touches nothing; a signal found within the base cadence of the previous cycle runs its cycle no sooner than the base cadence allows; an event that finds no signal file runs no cycle.
- The name-repair loop backs off while nothing drifted and resets after a repair.
- The readiness reading: ready on `idle-done`, `idle-stalled` of any cause, `asking`/`awaiting-input` and `busy`/`background` with a blank or absent box; held with a reason on `busy`/`turn`, an unmatched turn start, `asking`/`ask`, `dead`, a draft in the box, and a pane too short; `unknown` of any cause falls back to the pane probe; a `session-start` signal after a turn end leaves the reading ready; a pane running-turn needle beside a hook turn end reads ready and logs the disagreement.
- An unmatched turn start stops holding on a later turn end, on an interrupt the provider reports, and after the pane probe reads idle on every poll for the override, a busy poll restarting the count; the watcher's hold persists across its ticks.
- The orch watcher completes a compact cycle on the boundary and types the reload steps on the next tick whose readiness is ready, with the pane probe reading not idle; the same for an auto-compaction boundary read at a turn end.
- The draft case: with a draft in the input box at the boundary, the hook's verb writes a delivery mark naming the watcher's `PendingResume` before the operator's turn start, the color and plugins steps are held with reason `draft` and typed at the next ready tick, and the watcher skips its pointer step.
- The hook fallback: with no mark, a mark older than the boundary, or a mark whose text differs from `PendingResume`, the watcher types the pointer after the plugins step as today, also when a `session-start` event was recorded; the mark is removed when the reload ends.
- `lyx orch resume-context` prints, as the provider's additional-context JSON, `PendingResume` while the phase is `resuming` with one set, the resume prompt naming the last handoff while the phase is `compacting`, and the reload prompt otherwise, writing the delivery mark first; in every phase but `resuming` it renders the role file before the pointer, so a changed role stencil is in the file the pointer names; outside the prime, with no orch state, or on a render failure, it prints a JSON error and writes no mark.
- The hook command `orchcli` sets changes to the prime's directory before running the verb, so with the session's cwd in another worktree the verb still renders the prime's pointer.
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
- `SpawnWatchdog` already derives `fabricengine.HubScratchDir(hub)`; the discover signal's path can be derived beside it by the caller that builds the engine's geometry.
- The readiness reading can be built from `Runner.SessionState`, `unmatchedTurnStart`, `turnStartHold` and the provider's `InputBoxText`, which already exist; keeping `IdleProbe` keeps the `Session` seam's fakes in `orchengine` tests small.
- `orchengine`'s unit tests drive `Watcher` against a fake `Session`; the #481 scenario is one more table row: compaction boundary found, fake probe not idle, fake state `asking`/`awaiting-input`, expect the reload typed on that tick; the draft case is a second row with a delivery mark matching `PendingResume` and a draft reason, expecting color and plugins held and the pointer step skipped.
- `settingsHooks` in `internal/shuttleengine/claudeengine/settings.go` lists the recording hooks; `SessionStart` is one more entry, with the matcher on the orch's spec field only, and `recordingCommand` already builds the stamp-and-payload command.
- `startCompacting` sets `State.LastHandoff` before `/compact` is typed, so `lyx orch resume-context` can read the note path from the persisted state while the phase is `compacting`; `RenderResumePrompt` and `RenderReloadPrompt` are the two render functions it reuses.
- `ReloadSkipsSkills` already marks a reload after a compaction; reading the delivery mark inside `reloadStep`'s normalisation keeps it the one place that decides which step is next.
- `hubgeom`'s `sortSpawnOrder` is pure and already tested; only `runStartTime` changes.
- The measurement procedure can live as a short section in the benchmark report itself, so the next run repeats it.
