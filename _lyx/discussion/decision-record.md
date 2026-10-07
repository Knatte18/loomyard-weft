# Decision record — lifecycle-waits

## Goal

The orch and batten side of a run's lifecycle stops costing time and stops sending noise:

- An orch compact cycle reloads as soon as its compaction lands, instead of timing out after the handoff timeout.
- The orch hears each run state change once, from batten, within seconds, with a notice that says how old the state is and where the stop report is; the driver messages the orch only with a question.
- Batten's history records the child's state changes and nothing else, and its quiet notice reflects what the agent is doing, naming an API-error stall.
- Each batten the orch starts runs in its own tmux window in the orch's session, so the window list names every run in flight.

## Scope

In:

- `internal/orchengine`: the compacting phase's completion test, and a same-mode cycle request queued during a cycle.
- `internal/battenshed`, with its wiring in `internal/battencli` and `shedrecipe`'s InnerRun entry, and `contracts/recipes/batten-recipe.yaml`: a Run-Shed row that waits on the child instead of ticking, event-timed notices with their new fields, and a quiet notice judged from agent activity.
- `internal/shuttleengine` (provider-neutral surface) and `internal/shuttleengine/claudeengine` (transcript reading): an agent-activity reading of a worktree's live shuttle runs.
- `contracts/stencils/shed/shed-template-driver.md`: when a batten-watched driver messages its parent.
- `internal/reedengine` and the batten CLI: `lyx batten run <slug> --window`.
- `contracts/stencils/orch/orch-template-role.md`: the orch starts battens with `--window`.
- Docs in the same commit: `internal/orchengine/doc.go`, `internal/battenshed/doc.go`, the batten CLI's verb text, `internal/reedengine`'s doc for the window op, the recipe's header comment, and `docs/overview.md` if its batten or reed description changes.

Out:

- The step watchdog and the Windows verify tree kill (note `verify-watchdog`).
- Pair-create push retry and rollback (task `fabric-pushes`).
- `orch-owns-the-loop` (`lyx shed start`, a lyx watch, one land command, `--continue`) and `runs-overview` (per-run status lines, an event log window).
- Reed's multi-window anchoring of strands (`reed-multi-window`): only the batten process gets a window here.

## Decisions

### D1 — Compacting completes on a boundary found by `CompactedSince`

Decision: `tickCompacting` decides the compaction landed by asking `session.CompactedSince(turnEnd, st.PhaseEnteredAt)`, where `turnEnd` is the newest turn end the watcher holds (`st.ReadingTurnEnd`, which D2 keeps current).
A found boundary qualifies whatever follows it, any `TurnEndsAfter` count included.
On completion (idle probe passing, as today): the stored reading is `ContextTokens` of that turn end (the transcript's newest qualifying entry, which may be a post-compaction assistant entry), `cycle_count` increments, `CompactionBaseline` moves to the boundary's time, and the reload sequence starts.
A boundary at or before `PhaseEnteredAt` still never completes the phase; the timeout path is unchanged.

Rationale: `ContextTokens` reports only the transcript's newest qualifying entry, so any assistant entry after the boundary hides it.
On 2026-10-07 the operator's message queued during `/compact` was answered at once after the boundary, so the reading never showed `Compacted` and the cycle timed out (support log, Interview turn 2).
`CompactedSince` searches for the boundary itself, the way the auto-compaction path already does.

### D2 — Turn ends read during compacting update the reading

Decision: in the tick's event read, a turn end read while the phase is `compacting` is stored through `storeReading` like an idle turn end, so `ReadingTurnEnd` and the reading follow the session.
It does not feed the idle-phase auto-compaction detection (`lastTurnEnd` / `w.compactedAt` stay idle-only), so one boundary is never handled by both paths.

Rationale: a restarted watcher in `compacting` re-reads through `ReadingTurnEnd`; it must name a current turn end, and the context reading after the cycle must be the session's real one.

### D3 — A same-mode cycle request queued during a cycle is satisfied by it

Decision: when a cycle completes, a pending cycle request of the same mode (compact for a compact cycle, clear for a clear cycle) whose `RequestedAt` is at or after the cycle's `PhaseEnteredAt` is removed as satisfied, with an info log line naming it.
A request of the other mode, or one older than the cycle's entry, is left for the idle phase as today.
Bound: it removes only a request the operator made while that very cycle ran, for the same thing the cycle just did.

Rationale: with D1 a compact cycle completes in about a minute, so a `distill` asked for during it (as on 2026-10-07, asked because the operator thought the cycle had failed) would otherwise start a second compaction right after the first.

### D4 — Run-Shed waits on the child instead of ticking

Decision: the InnerRun row's `Call` no longer sleeps one poll interval and returns `Stuck` on every arm.
Each arm keeps its actions (spawn, adopt, revive, resume on a decision, done-grace) but waits inside the call, checking every `poll_interval_s`, and returns only when:

- the child's `State` differs from the state the call began with (the returned budget-exempt `Stuck`'s output names the change, for example `child running → blocked at Plan-Review`);
- the arm's own event fires: a decision record for an awaiting child (the resume runs, and the call returns once the child leaves `awaiting`), the driver strand ending or the grace elapsing for a done child (returns `Done`);
- batten's own status carries `pause_requested` (a budget-exempt `Stuck` saying so, so the engine pauses at the boundary);
- the context is cancelled, or a status read fails (a hard error, as today).

`poll_interval_s` becomes 2 in `batten-recipe.yaml`: it is now the in-call check interval, a stat of the child's status file plus a decode only when its modification time changed, so the row reacts within seconds.
Every `Stuck` the row returns is budget-exempt, so the 12-hour running watch window and its `max_bounces: 1440` rationale go; the recipe header comment is rewritten.
Notices (D5, D6) are evaluated on every check inside the wait, not only at call entry.
Bound: the row now waits on a running child without a time limit; what bounds it is `lyx batten pause` (honoured within one check), cancellation, and the quiet notice of D6 reaching the orch.

Rationale: the operator asked for no ticks (2026-10-07); a bounce every 30 s filled batten's history with `Run-Shed → stuck` entries (83 in an hour for `test-tiers`) and delayed every notice by up to 30 s.
A 2 s stat poll is portable and needs no new dependency.

### D5 — State-change notices: timing and content

Decision: batten sends the state-changed notice (child left running: blocked, paused, failed, awaiting, done) from inside the D4 wait, once the driver's stop report for that stop exists, or the driver strand has ended, or 3 minutes after the change, whichever comes first.
The stop report is the path the driver's park marker (`driver-parked`) holds; for a done child, whose driver ends its session instead of parking, the newest file in the run's drive-reports directory.
Every batten notice (state-changed, driver-dead, quiet, api-error) carries, after today's slug, condition, state and error:

- `since <time>`: when the child entered its current state, from the child's newest history entry's time (its `last_at` when folded);
- `history <n>`: the child's history length;
- `report <path>`: the stop report path, on a state-changed notice that has one; `report none yet` when the 3 minutes passed without one.

The attach command stays.
The 3-minute report wait is a constant.
Bound: the report wait delays a notice by at most 3 minutes; the episode rules (one notice per episode, the restart marker) are unchanged.

Rationale: the board asks that batten own state changes and carry the drive report path, which exists only after the driver writes it; `since` and `history` let the orch recognise a notice that went stale while it waited in the queue, without a status call.

### D6 — Quiet judged from agent activity; an API-error notice

Decision: `shuttleengine` gains a provider-neutral reading of a worktree's agent activity: for each live shuttle run under the worktree's run-directory root (a dead pid skipped, as the wait-marker reader does), the time of its agent's last activity (the newer of the transcript's last write and the run's last event) and whether the transcript's newest main-chain turn end is an API error, with its text.
`claudeengine` supplies the transcript part: the transcript path from the run's newest turn end, and an API-error turn end recognised by Claude's `isApiErrorMessage` marker on a main-chain assistant entry.
Batten reads it through an injected seam (wired in `battencli`, told the child worktree's paths).
For a running child with a live driver strand:

- `quiet`: no live run's agent has been active for `notice_quiet_min` and no Go-side wait marker (verify or shuttle) is live; the notice says how long the agents have been idle.
- `api-error`: the newest turn end of a live run is an API error and that agent has not been active for 2 minutes since; the notice names the producer and the error text, and takes precedence over `quiet`.

Episode keys move from the status file's modification time to the activity: a quiet or api-error episode ends when the agent is active again.
The status file's modification time no longer decides quiet.
Bound: both conditions are informational and change no outcome; with no live run found the quiet condition falls back to "no agent activity readable" after `notice_quiet_min` since the child's newest history entry.

Rationale: the status file changes only at a step's end, so a long live step tripped quiet, and on 2026-10-07 five runs stalled on an API error with notices arriving 45 minutes later saying nothing of the cause.
A live Go-side wait is not quiet: a gate is working, and a hung gate is `verify-watchdog`'s concern.

### D7 — A batten-watched driver messages its parent only with a question

Decision: the driver stencil renders its parent-notification rule by whether batten watches the run.
When it does, the driver sends the parent a message only for:

- an `awaiting` envelope's `parent_notice`, relayed verbatim as today;
- a stop report that asks the parent a question the driver cannot settle itself, the message naming the run-id, the report path and the question.

It never messages the parent that the run stopped, halted, finished, or that friction failed; those reach the orch as batten's D5 notice carrying the report path.
A run batten does not watch keeps today's rule.
Whether batten watches is decided in Go at render time from the run's seed (batten's Seed-Child records it), not by the agent.
Parent-review requests and escalation briefs from producer sessions are unchanged.
Bound: only the driver's "run stopped" message is removed, and only where a batten notice replaces it.

Rationale: on 2026-10-06 the orch got two messages for every halt, the batten one often stale; one channel per event, with batten owning state changes.

### D8 — `lyx batten run <slug> --window`

Decision: `lyx batten run <slug> --window` opens a tmux window named `batten:<slug>` in the reed session of the worktree the command runs in (the prime's, where the orch lives), running `lyx batten run <slug>` resolved through reed's pane-binary chokepoint, and returns at once with a JSON envelope naming the window.
The window is created through a new reed op; the batten process in it is outside the caller's process group, so it survives the orch's shell.
When a `batten:<slug>` window already exists with a live process, the verb succeeds without starting a second one and says so; a window whose process has exited is replaced.
The window closes when batten exits.
With no reed session for the worktree the verb refuses, naming `lyx orch start` (or plain `lyx batten run <slug>`) as the way forward, with a refusal-spec row and a test.
The orch role stencil replaces its `setsid nohup` instruction with `lyx batten run <slug> --window`.
Bound: the flag only chooses where the same `lyx batten run` executes; it never changes the run, and it never starts a duplicate for a slug whose window is live.

Rationale: the board's window-per-batten section (operator idea, 2026-10-03); a window per run makes the tmux window list the list of runs in flight.

## Constraints

- `PATTERN-orch-pane-single-writer`: batten notices still go through `orchengine.QueueNotice`; only the watcher types them.
- `PATTERN-told-geometry`: `battenshed` takes the child's paths and the activity and report readers as injected seams; it imports no `lyxcwd`.
- `PATTERN-shuttle-provider-seam`: transcript and `isApiErrorMessage` knowledge stays in `claudeengine`; batten and `shuttleengine`'s public reading are provider-neutral.
- `PATTERN-wait-mark-display-only`: D6 reads the wait marker files, never the `@lyx_wait` pane options.
- `PATTERN-fabric-vocabulary`: `battenshed` text says "the task worktree" and "the pair", never the side names.
- `PATTERN-shed-producer-seam` and `PATTERN-shed-verb-set`: D4 changes only the InnerRun producer and its recipe config; shedengine's bounce and exempt-Stuck semantics are unchanged.
- `PATTERN-pane-binary-resolution`: the D8 window resolves `lyx` through `panebin.go`.
- `PATTERN-shell-mechanics-seam`: the window's command string is built through `internal/shell`.
- `PATTERN-spawn-observability`: the window's process start is logged through `internal/logger`.
- `PATTERN-cli-cobra` and `PATTERN-refusal-way-forward`: `--window` is a flag on the existing verb; its refusal names the way forward, lands its spec row and has a test.
- `PATTERN-parent-directive`: the driver stencil still renders the parent directive; D7 changes only when it sends.
- `PATTERN-driver-choice-single-site`: D7 reads the batten-watched fact from the seed at render time and does not re-read the seed driver.
- `PATTERN-test-speed`: the D4 wait, the 3-minute report wait and D6's windows run on an injected clock and sleep; D8's window test is in the `tmux` tier.
- `PATTERN-comment-line-breaks`: every changed Go comment uses semantic line breaks.

## Auto-mode assumptions

- A1: the 2026-10-07 transcript shows the compaction cause: the boundary at 10:06:41Z was followed at 10:06:42Z by a queued operator message and its answer, in the same transcript file.
- A2: a 2 s stat poll is "event-driven" enough; no file-notification dependency.
- A3: dropping the 12-hour running watch window is acceptable, since the quiet notice and `lyx batten pause` cover a run nobody watches.
- A4: 3 minutes is enough for a driver to write its stop report and park.
- A5: an API-error stall is notified after 2 minutes without agent activity; quiet stays at `notice_quiet_min: 45`.
- A6: Claude's `isApiErrorMessage` marker on a synthetic main-chain assistant entry identifies an API-error turn end (seen in a 2026-10 transcript).
- A7: the orch runs inside the prime's reed session, so "the reed session of the worktree the command runs in" is the orch's session.
- A8: D3's same-mode satisfaction is part of "the reload starts on time", the board's side effect.

## Open risks

- Claude Code's transcript format is internal; D1 and D6 rely on the `compact_boundary` entry and the `isApiErrorMessage` marker.
- A notice still reaches the orch only when its turn ends; `since` and `history` make staleness recognisable but do not prevent it.
- A driver that writes no stop report and stays alive delays its state-changed notice by the full 3 minutes.
- The tmux window lives only as long as the tmux server; a tmux restart kills the batten processes in it, as it already kills the strands.
- D7 relies on the driver stencil being followed; a driver that still messages "the run stopped" is noise, not a failure.

## Acceptance criteria

- A compact cycle whose transcript holds a boundary after the phase entry followed by an assistant turn end completes on the next tick with the idle probe passing: the reload starts, `cycle_count` increments, `CompactionBaseline` is the boundary's time, and no `compaction timed out` abort is recorded.
- A compact cycle whose boundary has no turn end after it completes the same way.
- A boundary at or before the phase entry does not complete the cycle, and the timeout abort still fires past the handoff timeout.
- A watcher restarted in `compacting` after a post-boundary turn end completes without typing `/compact` again.
- A compact request made during a compact cycle is removed as satisfied when the cycle completes; a clear request made during it stays pending.
- A child that runs for many check intervals adds no batten history entry; its change to `blocked` adds exactly one budget-exempt entry naming the change, with its time.
- A pause requested on batten while the child runs takes effect within one check interval.
- An awaiting child's decision record is acted on within one check interval; a done child's row returns `Done` once its driver strand ends or the grace elapses.
- A halted child whose driver parks with a stop report produces one notice carrying `since`, `history` and that report path; a child whose driver writes no report produces one notice after 3 minutes with `report none yet`.
- A running child whose producer's transcript keeps growing gets no quiet notice however long its status file is unchanged; one whose agents are idle for `notice_quiet_min` with no live wait marker gets one; a live verify or shuttle wait marker suppresses it.
- A running child whose producer's newest turn end is an API error and whose agent is then idle for 2 minutes gets one `api-error` notice naming the producer and the error text; activity afterwards ends the episode.
- The driver stencil rendered for a batten-watched run has no instruction to message the parent on a stop, halt, done or friction failure, and keeps the `parent_notice` relay and the question message; rendered for an unwatched run it keeps today's rule.
- `lyx batten run <slug> --window` creates a `batten:<slug>` window in the orch's reed session running the batten and returns at once; a second call while it is live starts nothing and says so; a call with no reed session refuses with its way forward.
- `go test ./...` and the `tmux`-tier tests are green.

## Notes for the plan writer

- Compaction: `tickCompacting` and the event loop in `tick()` in `internal/orchengine/watcher.go`; the `CompactedSince` seam exists on the watcher's session interface; `storeReading` records `ReadingTurnEnd`; stale request removal is in `tickIdle`.
- Run-Shed: `internal/battenshed/innerrun.go` (`Call`, `exemptWait`, `callAwaiting`, `callDone`, `callHalted`), notices in `notice.go`, deps in `deps.go`, config read in `internal/shedrecipe/entries_batten.go`.
- Notice queue: `internal/orchengine/notice.go`, wired from `internal/battencli/wire.go`.
- Wait marker readers: `verifytree.ReadMarker` and `shuttleengine.ReadWaitMarker`, composed in `internal/loomcli/reviewstatus.go`.
- Driver stencil: `## Notifying the parent` and `## Stopping on a non-running state` in `contracts/stencils/shed/shed-template-driver.md`; the park marker is written by the park command.
- Orch stencil: the start step in `contracts/stencils/orch/orch-template-role.md`.
