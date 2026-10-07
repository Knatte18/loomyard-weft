# Decision record — lifecycle-waits

## Goal

The orch and batten side of a run's lifecycle stops costing time and stops sending noise:

- An orch compact cycle reloads as soon as its compaction lands, instead of timing out after the handoff timeout.
- The orch hears each run stop once, from batten, within seconds, with a notice that says how old the state is and where the stop report is; the exception is an awaiting stop that carries a `parent_notice`, which the driver relays, and which batten announces itself only when no relay can have happened or the stop is still undecided 10 minutes after its report.
  The driver messages the orch only with that relay, a question, or a `done` stop whose friction failed.
- Batten's history records the child's state changes, a requested pause and a failed read of the child's status file, nothing else, and its quiet notice reflects what the agent is doing, naming an API-error stall.
- Each batten the orch starts runs in its own tmux window in the orch's session, so the window list names every run in flight.

## Scope

In:

- `internal/orchengine`: the compacting phase's completion test and a same-mode cycle request queued during a cycle.
- `internal/battenshed`, with its wiring in `internal/battencli` and `shedrecipe`'s InnerRun entry, and `contracts/recipes/batten-recipe.yaml`: a Run-Shed row that waits on the child instead of ticking, event-timed notices with their new fields, and a quiet notice judged from agent activity.
- `internal/shuttleengine` (provider-neutral surface) and `internal/shuttleengine/claudeengine` (transcript reading): an agent-activity reading of a worktree's live shuttle runs.
- `contracts/stencils/shed/shed-template-driver.md`, its render in `internal/loomcli`, and the watched marker's path in `internal/shedrun`: when a batten-watched driver messages its parent.
- `internal/reedengine` and the batten CLI: `lyx batten run <slug> --window`, and reed addressing its strand window by id.
- `contracts/stencils/orch/orch-template-role.md`: the orch starts battens with `--window`.
- Docs in the same commit: `internal/orchengine/doc.go`, `internal/battenshed/doc.go`, the batten CLI's verb text, `internal/reedengine`'s doc for the window op and the strand-window addressing, the recipe's header comment, and `docs/overview.md` if its batten or reed description changes.

Out:

- The step watchdog and the Windows verify tree kill (note `verify-watchdog`).
- Pair-create push retry and rollback (task `fabric-pushes`).
- `orch-owns-the-loop` (`lyx shed start`, a lyx watch, one land command, `--continue`) and `runs-overview` (per-run status lines, an event log window).
- Reed's multi-window anchoring of strands (`reed-multi-window`): only the batten process gets a window here, and no strand is placed in a window of its own.

## Decisions

### D1 — Compacting completes on a boundary found by `CompactedSince`

Decision: `tickCompacting` decides the compaction landed by asking `session.CompactedSince(turnEnd, st.PhaseEnteredAt)`, where `turnEnd` is the turn end the watcher holds (`st.ReadingTurnEnd`).
A found boundary qualifies whatever follows it, any `TurnEndsAfter` count included.
On completion (idle probe passing, as today): the stored reading is `ContextTokens` of that turn end (the transcript's newest qualifying entry, which may be a post-compaction assistant entry), `cycle_count` increments, `CompactionBaseline` moves to the boundary's time, and the reload sequence starts.
A boundary at or before `PhaseEnteredAt` still never completes the phase; the timeout path is unchanged.

Rationale: `ContextTokens` reports only the transcript's newest qualifying entry, so any assistant entry after the boundary hides it.
On 2026-10-07 the operator's message queued during `/compact` was answered at once after the boundary, so the reading never showed `Compacted` and the cycle timed out (support log, Interview turn 2).
`CompactedSince` searches for the boundary itself, the way the auto-compaction path already does.

### D2 — A same-mode cycle request queued during a cycle is satisfied by it

Decision: the cycle's entry into the handoff-requested phase already removes any request pending when it starts, as today, so every request found later was made after the cycle started; no cycle start time is kept, since `PhaseEnteredAt` is overwritten by every later phase and the entry's own clear is what tells the cycle's trigger from a later request.
A pending cycle request of the same mode (compact for a compact cycle, clear for a clear cycle) is removed as satisfied, with an info log line naming it, at the moment the cycle's own action takes effect:

- compact: when the compaction completes (D1), if the request's `RequestedAt` is not after the boundary's time;
- clear: when the `/clear` is confirmed typed.

A request of the other mode and a compact request made after the boundary are left for the idle phase as today.
Bound: it removes only a request made after the cycle started and before the cycle's action took effect, for the same thing the action did.
The request time (written by the CLI) and the boundary time (the transcript's) are wall clock on one host.

Rationale: with D1 a compact cycle completes in about a minute, so a `distill` asked for during it (as on 2026-10-07, asked because the operator thought the cycle had failed) would otherwise start a second compaction right after the first.
Satisfying the request is part of "the reload starts on time", the board's side effect.

### D3 — Run-Shed waits on the child instead of ticking

Decision: the InnerRun row's `Call` no longer sleeps one poll interval and returns `Stuck` on every arm.
Each arm keeps its actions (spawn, adopt, revive, resume on a decision, done-grace) but waits inside the call, checking every `poll_interval_s`, and returns only when:

- the child's `State` differs from the state the call began with (the returned budget-exempt `Stuck` carries the change, for example `child running → blocked at Plan-Review`, in the pointer's `Path`: a history entry persists the path and not the reason, and entries with equal output fold, so a distinct text per change is what keeps each change its own entry; the reason mirrors it for `lyx batten status`);
- the arm's own event fires: a decision record for an awaiting child (the resume runs, and the call returns once the child leaves `awaiting`), the driver strand ending or the grace elapsing for a done child (returns `Done`);
- batten's own status carries `pause_requested`, read through an injected seam told batten's own status path (a budget-exempt `Stuck` whose `Path` is `pause requested at <time>`, the time being the check's, so a second pause is an entry of its own and not a fold; the engine pauses at the boundary; a history entry that is not a state change);
- the stat of the child's status file fails: the call returns a budget-exempt `Stuck` naming the file, and the next call's status read decides, an absent file by the row's absent-file handling as today and an unreadable one as the hard error (the other history entry that is not a state change);
- the context is cancelled, or decoding the child's status file fails (a hard error, as today).

`poll_interval_s` becomes 2 in `batten-recipe.yaml`: it is now the in-call check interval, a stat of the child's status file plus a decode only when its modification time changed, so the row reacts within seconds.
The baseline modification time is stat'ed before the status read that defines the state the call began with (the spawn arm's second read, after the spawn, included), and any difference from it, not only a later time, triggers the decode, so a write landing between that read and the first check is seen.
The decode also runs at every `notice_probe_s` check whatever the modification time says, so a write that left the time unchanged (two writes in one timestamp tick) is seen within that interval.
Every `Stuck` the row returns is budget-exempt, so no Stuck is counted: the 12-hour running watch window and the row's `max_bounces: 1440` go, and the recipe header comment is rewritten.
A pending notice (D4) is derived from the child's status and the episode marker of sent notices, so it outlives the call that saw the change, and it is checked on every check inside the wait.
The wait sends it when its own trigger is met; a call that returns for the state change itself leaves it to the wait of the call that begins in the new state, and a pause or error return leaves it pending.
The one return that ends batten, `Done`, makes a last send attempt first; a notice not queued then is logged once and lost.
The cheap file reads (the status stat, the pause flag, the decision record, the report stat) run on every check.
Every read that costs a process or a multiplexer round trip runs at most once per `notice_probe_s`, a new recipe config key set to 30, not on every 2 s check: driver liveness (a reed status call) wherever it is read, a running child's agent activity and wait markers (D5), the status decode that ignores the modification time, and the retry of a resume that the child's driver refused as not parked.
That retry logs once per episode and goes on until the driver parks or the child's state changes.
A done child's `Done` and a pending notice's driver-ended trigger therefore follow the driver's end within `notice_probe_s`.
Bound: the row now waits on a running child without a time limit; what bounds it is `lyx batten pause` (honoured within one check), cancellation, and the notices of D4 and D5 reaching the orch, with the limits D5 states.
A running child whose driver strand is dead and whose status file does not change returns nothing from the wait; the driver-dead notice is the only signal.

Rationale: the operator asked for no ticks (2026-10-07); a bounce every 30 s filled batten's history with `Run-Shed → stuck` entries (83 in an hour for `test-tiers`) and delayed every notice by up to 30 s.
A 2 s stat poll is portable and needs no new dependency.

### D4 — State-change notices: timing and content

Decision: batten sends the state-changed notice (child left running: blocked, paused, failed, awaiting, done) from inside the D3 wait.
For a parked stop (blocked, paused, failed, awaiting) it goes once the driver's stop report for that stop exists, or the driver strand has ended, or 3 minutes after the change, whichever comes first; the 3 minutes count from `since`.
A done child's notice goes on the first check that sees it done, with no report: Worktree-Teardown removes the task worktree the path would point into right after `Done`, so the orch could not read it.
An `awaiting` child whose status carries a non-empty `parent_notice` gets no state-changed notice while the driver's relay of it (D6) can still happen and the watched marker (D6) stands for this batten, the only case where the driver's parent is the notice's destination.
Batten sends it when any of these holds:

- the driver strand has ended;
- the 3 minutes passed with no stop report, since the relay follows the report and neither can then have happened;
- the child is still awaiting with no decision record 10 minutes after the stop report's file time, whatever the driver's state.

Bound: a relay that a live, reported driver failed or skipped delays the stop's announcement by at most those 10 minutes and never loses it; a relay that succeeded, followed by a decision slower than that, costs one more notice, never a loss.
An `awaiting` child whose status carries none (every PR-Gate hand-off, and a Bouncer escalation whose notice failed to render), and one whose batten holds no watched marker for the run, gets the state-changed notice at once like any parked stop, since nothing else announces it to the notice's destination.
The stop report is the path the driver's park marker (`driver-parked`) holds.
A report counts only when its file time (the marker's) is at or after `since`, the time batten first saw the child in its state; an older one is an earlier stop's and never qualifies.
Every batten notice (state-changed, driver-dead, quiet, api-error) carries, after today's slug, condition, state and error:

- `since <time>`: the status file's modification time at the first sight of the child in its state, taken by a check inside the wait or by the baseline stat of a call that begins in a new state; batten's in-process episode state holds it across the calls, pause returns and error returns that follow, until the child's state changes, so a rewrite of the file in the same state does not move it, and only a batten start reads it from the file, so a restart that finds the file rewritten in the same state may send a notice again; the child's history cannot supply it, since a pause or a producer failure appends no entry;
  the episode key of a state-changed and of a driver-dead notice is the state and `since`, and a quiet or api-error notice carries `since` as content only, its key being D5's activity;
- `history <n>`: the child's history length;
- `report <path>`: the stop report path, on a state-changed notice for a parked stop that has one; `report none yet` when the 3 minutes passed without one; no `report` on a done notice.

The attach command stays.
The 3-minute report wait and the 10-minute relay wait are constants.
A notice is recorded as sent only when `QueueNotice` reported it queued; the `Notify` seam reports that.
A notice is queued only while the orch state records a strand, the same fact D6's marker tests, read through one injected seam: with none recorded batten does not call `QueueNotice`, logs one Warn per episode, and tries again when a strand is recorded, checked once per `notice_probe_s`, until the episode ends or the `Done` return makes its last attempt.
A notice whose queueing failed with an error is retried at most once per `notice_probe_s` and at most 3 times, then dropped with a Warn, so a stop nobody can receive neither retries nor logs without end.
A batten started again sends the notices not yet sent, because the episode marker records only sent ones.
Bound: the report wait delays a notice by at most 3 minutes; the episode rules (one notice per episode, the marker of sent keys) are unchanged apart from the keys above.

Rationale: the board asks that batten own state changes and carry the drive report path, which exists only after the driver writes it; `since` and `history` let the orch recognise a notice that went stale while it waited in the queue, without a status call.
`since` comes from the status file's modification time because the child's history has no time for a pause or a producer failure, and an older newest-entry time would start the 3 minutes at once and let an earlier stop's report qualify.
With D6 the driver no longer backs batten up, so a notice lost to a full or absent queue would leave the run unannounced.

### D5 — Quiet judged from agent activity; an API-error notice

Decision: `shuttleengine` gains a provider-neutral reading of a worktree's agent activity: for each live shuttle run under the worktree's run-directory root (a dead pid skipped, as the wait-marker reader does), the time of its agent's last activity (the newer of the transcript's last write and the run's last event) and whether the transcript's newest main-chain turn end is an API error, with its text.
`claudeengine` supplies the transcript part: the transcript path from the run's newest turn end, and an API-error turn end recognised by Claude's `isApiErrorMessage` marker on a main-chain assistant entry.
Batten reads it through an injected seam (wired in `battencli`, told the child worktree's paths).
For a running child with a live driver strand:

- `quiet`: no live run's agent has been active for `notice_quiet_min` and no Go-side wait marker (verify or shuttle) is live; the notice says how long the agents have been idle.
- `api-error`: the newest turn end of a live run is an API error and that agent has not been active for 2 minutes since; the notice names the producer and the error text, and takes precedence over `quiet`.
  The error text is cut to one line and 200 characters, as the child's error is, and the line keeps the fixed `[batten notice]` prefix.

Episode keys move from the status file's modification time to the activity: a quiet or api-error episode ends when the agent is active again.
The status file's modification time no longer decides quiet.
Bound: both conditions are informational and change no outcome, and each has stated limits:

- with no live run found, quiet falls back to "no agent activity readable" after `notice_quiet_min` since the later of the child's newest history entry and `since` (D4), since a pause or a producer failure appends no history entry, which is also its episode key;
- a live run whose transcript is unreadable is judged by its last event, or by its start when it has none;
- with several live runs, quiet needs every run idle and api-error needs any one run in error;
- a dead pid is skipped, so an API error in a run whose agent died is not reported;
- a run that keeps writing without finishing its step, or hangs inside a live wait marker, is never quiet: only `lyx batten status` and `lyx batten pause` stop it, the hung gate being `verify-watchdog`'s concern.

Rationale: the status file changes only at a step's end, so a long live step tripped quiet, and on 2026-10-07 five runs stalled on an API error with notices arriving 45 minutes later saying nothing of the cause.
A live Go-side wait is not quiet: a gate is working.

### D6 — A batten-watched driver messages its parent only with a question

Decision: the driver stencil renders its parent-notification rule by whether batten watches the run.
When it does, the driver sends the parent a message only for:

- an `awaiting` envelope's `parent_notice`, relayed verbatim as today;
- a stop report that asks the parent a question the driver cannot settle itself, the message naming the run-id, the report path and the question;
- a `done` stop whose envelope reports `friction: failed`, as today, since batten's done notice carries no report and does not know the friction result.

It never messages the parent that the run stopped, halted or finished, nor sends the generic escalation line at a stop, an `awaiting` stop with no `parent_notice` included; those reach the orch as batten's D4 notice.
A run batten does not watch keeps today's rule.
Whether batten watches is decided in Go at render time, not by the agent: batten writes a `batten-watched` marker holding its pid in the run's scratch directory (its path declared by `shedrun` beside the park marker, written when Seed-Child seeds the run and rewritten by each batten that takes the run over), and the render counts the run as watched only while that pid is live.
Batten writes the marker only while its notices have a destination that is also the driver's parent: it runs in the prime, whose orch its notices name, the run's own parent (the orch of the worktree the pair was created from, as the driver is told it) is that prime orch, and the orch state records a strand; otherwise it removes it.
A batten started from another worktree writes none, since its notices still go to the prime's orch while its driver's parent is that worktree's orch, so its driver keeps today's rule.
A prime batten that adopts or revives a pair created from another worktree, or one with no resolvable parent, writes none either: the fresh launch of a revive then renders today's rule, and the relay (or its absence) is that parent's, not the prime orch's.
The seed is not used: it is write-once and compared for equality against what the child's own bootstrap writes, so an extra param makes that bootstrap refuse.
Parent-review requests and escalation briefs from producer sessions are unchanged.
Bound: only the driver's "run stopped" message is removed, and only where a live batten with a notice destination replaces it at render time.
The rule is fixed at the driver's fresh launch, since a resume types one line into the live driver and renders nothing: a driver launched unwatched and adopted by batten keeps today's rule (a duplicate message, never a loss), and one launched watched keeps the silent rule whatever happens to batten.
A batten that dies after the render, or an orch strand that disappears after it, leaves the run's stops unannounced until batten runs again (Open risks).

Rationale: on 2026-10-06 the orch got two messages for every halt, the batten one often stale; one channel per event, with batten owning state changes and the driver keeping the awaiting hand-off text only it holds.

### D7 — `lyx batten run <slug> --window`

Decision: `lyx batten run <slug> --window` opens a tmux window named `batten:<slug>` in the reed session of the worktree the command runs in (the prime's, where the orch lives), running `lyx batten run <slug>` resolved through reed's pane-binary chokepoint, and returns at once with a JSON envelope naming the window.
The window is created through a new reed op, detached and never selected, so it takes no focus; the id tmux returns is what every later op addresses it by, the name being for display.
Reed learns its strand window from the window holding its Selvage pane: reed state records the pane's id, and each op resolves that pane's window id through the multiplexer.
Every op that today targets the session's current window goes through that one addressing seam, which replaces `exactSessionWindowTarget` and is given to the engine-less pane listing as its target: pane enumeration, layout and split, and also the window-size, resize-hook and attach-geometry reads.
The pane-generation read stays session-scoped and outside the seam: it needs only session fields, so it depends on none of the ids the seam resolves.
It validates the recorded ids but fails open when its probe fails, so the seam does not rely on it alone: a recorded id counts as present only when it appears in the session's own pane listing, which carries each pane's window id, and never through a server-wide pane target, since pane ids are server-wide and a stale id can name another session's pane.
An id absent from that listing is treated as absent.
A session whose recorded Selvage pane is present in that listing, alive or a dead corpse, resolves that pane's window; with none, the window of any present recorded strand pane; and only with none of those the seam falls back to today's current-window target.
While the session has a present Selvage or strand pane, a batten window, current or not, is therefore never read as the strands' panes and its pane is never reaped as untracked.
With neither, as at the session's first `up`, no strand window exists yet to protect, and a Selvage pane recreated into a current batten window defines the strand window from then on.
`new-window` joins the verbs the capability probe requires, since the op cannot degrade; a multiplexer without it, and a `new-window` call that fails, refuse the verb naming `lyx batten run <slug>` in a terminal as the way forward.
The batten process in the window is outside the caller's process group, so it survives the orch's shell, but not the end of the reed session or a tmux restart.
The op creates the window and sets `remain-on-exit off` for it in one multiplexer invocation, so it closes when batten exits, even when batten exits at once.
The chained `set-option -w` targets the window by its exact name (`=<session>:=batten:<slug>`), because the id is known only when the invocation returns and an option chained after a detached `new-window` without a target lands on the session's current window, the strands' window, whose `remain-on-exit` must stay on.
After the invocation the op reads `remain-on-exit` back from the new window by the id the invocation printed.
A window that does not read `off`, for example one created under a name two concurrent calls share, where the chained option lands on the older window and does not fail, is killed by that id, and the verb refuses naming the failure and `lyx batten status <slug>` as the way forward, since the run may already be live from the call that kept the older window; `lyx batten run <slug>` in a terminal follows only when it shows none.
When a `batten:<slug>` window already exists with a live process, the verb succeeds without starting a second one and says so; a dead pane found under that name is replaced.
With no reed session for the worktree the verb refuses, naming `lyx orch start` (or plain `lyx batten run <slug>`) as the way forward, with a test.
The orch role stencil replaces its `setsid nohup` instruction with `lyx batten run <slug> --window`.
Bound: the flag only chooses where the same `lyx batten run` executes; it never changes the run.
A second batten for one slug is refused by the run's own lock, not by the window check: two calls in the same moment may open two windows, the second batten exits on the lock, and the op's read-back kills the window that does not read `off`, so no dead window stays.
The verb's envelope reports a started window only, never a started batten; the lock refusal of a window that closed is read from batten's own log.

Rationale: the board's window-per-batten section (operator idea, 2026-10-03); a window per run makes the tmux window list the list of runs in flight.
Reed sets `remain-on-exit on` for every session and enumerates the current window only, so both need a stated answer once a second window exists.

## Constraints

- `PATTERN-orch-pane-single-writer`: batten notices still go through `orchengine.QueueNotice`; only the watcher types them.
- `PATTERN-told-geometry`: `battenshed` takes the child's paths, batten's own status path, and the activity, report, notice and watched-marker seams as injected ones; it imports no `lyxcwd`.
- `PATTERN-shuttle-provider-seam`: transcript and `isApiErrorMessage` knowledge stays in `claudeengine`; batten and `shuttleengine`'s public reading are provider-neutral.
- `PATTERN-wait-mark-display-only`: D5 reads the wait marker files, never the `@lyx_wait` pane options.
- `PATTERN-fabric-vocabulary`: `battenshed` text says "the task worktree" and "the pair", never the side names.
- `PATTERN-shed-producer-seam` and `PATTERN-shed-verb-set`: D3 changes only the InnerRun producer and its recipe config; shedengine is unchanged, its bounce and exempt-Stuck semantics included, and the row no longer returns a counted Stuck.
- `PATTERN-pane-binary-resolution`: the D7 window resolves `lyx` through `panebin.go`.
- `PATTERN-shell-mechanics-seam`: the window's command string is built through `internal/shell`.
- `PATTERN-spawn-observability`: the window's process start is logged through `internal/logger`.
- `PATTERN-cli-cobra` and `PATTERN-refusal-way-forward`: `--window` is a flag on the existing verb; `refusal-spec.md` lists batten refusals out of scope and has no batten section, so no spec row lands, but every refusal of the verb (no reed session, a multiplexer without `new-window`, a failed `new-window`, the read-back) names its way forward and a test reaches each.
- `PATTERN-parent-directive`: the driver stencil still renders the parent directive; D6 changes only when it sends.
- `PATTERN-driver-choice-single-site`: D6 reads the watched marker at render time and does not re-read the seed driver.
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: the watched marker is ephemeral, under `.lyx`, and its path is declared in `shedrun`.
- `PATTERN-test-speed`: the D3 wait, the 3-minute report wait and D5's windows run on an injected clock and sleep; D7's window test is in the `tmux` tier.

## Auto-mode assumptions

- A1: a 2 s stat poll is "event-driven" enough; no file-notification dependency.
- A2: dropping the 12-hour running watch window is acceptable, since the notices, `lyx batten status` and `lyx batten pause` cover a run nobody watches, with the limits D5 states.
- A3: 3 minutes is enough for a driver to write its stop report and park.
- A4: an API-error stall is notified after 2 minutes without agent activity; quiet stays at `notice_quiet_min: 45`.
- A5: Claude's `isApiErrorMessage` marker on a synthetic main-chain assistant entry identifies an API-error turn end (seen in a 2026-10 transcript).
- A6: the orch runs inside the prime's reed session, so "the reed session of the worktree the command runs in" is the orch's session.
- A7: a batten run from the prime over a pair created from the prime is the case the Goal describes; its notices and its driver's parent are the same orch.

## Open risks

- Claude Code's transcript format is internal; D1 and D5 rely on the `compact_boundary` entry and the `isApiErrorMessage` marker.
- A notice still reaches the orch only when its turn ends; `since` and `history` make staleness recognisable but do not prevent it.
- A driver that writes no stop report and stays alive delays its state-changed notice by the full 3 minutes.
- A batten that dies after its driver rendered, or an orch strand that disappears after it, leaves the run's stops unannounced until batten runs again, which then sends the notices not yet sent.
- The tmux window lives only as long as the reed session; ending the session or restarting tmux kills the batten processes in it, as it already kills the strands, which the earlier `setsid nohup` start survived.
- A done notice not queued at the `Done` return, the last send attempt, is lost: batten ends and nothing retries it.
- D6 relies on the driver stencil being followed; a driver that still messages "the run stopped" is noise, not a failure.
- An awaiting stop with a `parent_notice` whose driver wrote its report, is alive and omitted or failed the relay is announced by batten only 10 minutes after the report, if the child is still undecided by then.
- A batten started from a worktree other than the prime leaves its driver on today's rule, so a run stop may reach the orch twice, never not at all.

## Acceptance criteria

- A compact cycle whose transcript holds a boundary after the phase entry followed by an assistant turn end completes on the next tick with the idle probe passing: the reload starts, `cycle_count` increments, `CompactionBaseline` is the boundary's time, and no `compaction timed out` abort is recorded.
- A compact cycle whose boundary has no turn end after it completes the same way.
- A boundary at or before the phase entry does not complete the cycle, and the timeout abort still fires past the handoff timeout.
- A watcher restarted in `compacting` after a post-boundary turn end completes without typing `/compact` again.
- A compact request made after a compact cycle started and before its boundary is removed as satisfied when the compaction completes, also when it was made during the handoff phase; a compact request made after the boundary, and a clear request made during the cycle, stay pending.
- A clear request made after a clear cycle started and before its `/clear` was confirmed is removed as satisfied; a compact request made during it stays pending.
- A child that runs for many check intervals adds no batten history entry; its change to `blocked` adds exactly one budget-exempt entry whose output names the change, and a later change back to `running` is an entry of its own, not a fold of the first.
- A child whose status file is rewritten between the call's status read and its first check is seen to have changed on that check; a write that leaves the modification time unchanged is seen within `notice_probe_s`.
- A pause requested on batten while the child runs takes effect within one check interval, adding one history entry; a second pause requested later, with no state change between, adds an entry of its own.
- A failed stat of the child's status file ends the wait, adding one history entry, and the next status read handles an absent file as today and fails hard on an unreadable one.
- An awaiting child's decision record is acted on within one check interval; a done child's row returns `Done` once its driver strand ends, seen within `notice_probe_s`, or the grace elapses, after one last send attempt of its notice.
- A done child's notice goes out on the first check that sees it done and carries no report path.
- A halted child whose driver parks with a stop report produces one notice carrying `since`, `history` and that report path, sent by the wait of the call that begins in the new state; a child whose driver writes no report produces one notice 3 minutes after the change with `report none yet`; a report written before the state change is never carried.
- A paused or failed child, whose history has no entry for the change, gets its notice's `since` and 3 minutes from the status file's modification time at the first sight of the state, and an earlier stop's report is not carried; a rewrite of the file in the same state, landing between two calls, moves neither `since` nor the episode key and sends no second notice.
- An awaiting child whose status carries no `parent_notice` produces one batten notice with its report path; one whose status carries a `parent_notice` produces none from batten while its driver is alive, a report exists and this batten holds the watched marker, and one notice when its driver strand has ended, 3 minutes passed with no report, or the child is still awaiting with no decision 10 minutes after the report; a batten that holds no watched marker for the run produces the notice as for a child with no `parent_notice`.
- A notice that `QueueNotice` queued is recorded as sent; with no strand recorded in the orch state `QueueNotice` is not called and one Warn is logged per episode; a notice whose queueing failed is sent again no more often than once per `notice_probe_s`, at most 3 times, and a batten started again sends a notice not yet sent.
- A running child whose producer's transcript keeps growing gets no quiet notice however long its status file is unchanged; one whose agents are idle for `notice_quiet_min` with no live wait marker gets one; a live verify or shuttle wait marker suppresses it.
- A running child whose producer's newest turn end is an API error and whose agent is then idle for 2 minutes gets one `api-error` notice naming the producer and the error text on one line; activity afterwards ends the episode.
- The driver stencil rendered for a run with a live batten's watched marker has no instruction to message the parent on a stop, halt or done, and no generic escalation line, and keeps the `parent_notice` relay, the question message and the message for a `done` stop whose friction failed; rendered for an unwatched run, or one whose marker names a dead pid, it keeps today's rule; the child's seed is unchanged.
- Batten writes the watched marker only while it runs in the prime, the run's parent is the prime's orch and the orch state records a strand, and removes it otherwise; a prime batten over a pair created from another worktree writes none, its driver keeps today's rule and its awaiting stop with a `parent_notice` gets batten's notice at once.
- `lyx batten run <slug> --window` creates a detached `batten:<slug>` window in the orch's reed session running the batten and returns at once; the window closes when batten exits, and the session's other windows keep `remain-on-exit on`, read back on both; a second call while it is live starts nothing and says so; two calls opening windows of one name in the same moment leave one window and no dead one, the call whose window was killed refusing with `lyx batten status <slug>` as its way forward; a call with no reed session, a multiplexer without `new-window` and a failed `new-window` each refuse with their way forward.
- A reed op run while the batten window is the session's current window leaves the strands untouched and the batten pane unreaped, the window-size reads included; a session whose Selvage pane is a dead corpse resolves its strand window through it, one with no Selvage pane resolves through a present strand pane, and with none keeps today's current-window targeting; a recorded pane id that names a pane of another session on the same server counts as absent; the pane-generation read targets the session.
- `go test ./...` and the `tmux`-tier tests are green.

## Notes for the plan writer

- Compaction: `tickCompacting` and the event loop in `tick()` in `internal/orchengine/watcher.go`; the `CompactedSince` seam exists on the watcher's session interface; stale request removal is in `tickIdle`.
- Run-Shed: `internal/battenshed/innerrun.go` (`Call`, `exemptWait`, `callAwaiting`, `callDone`, `callHalted`), notices in `notice.go`, deps in `deps.go`, config read in `internal/shedrecipe/entries_batten.go`.
- Notice queue: `internal/orchengine/notice.go`, wired from `internal/battencli/wire.go`.
- Wait marker readers: `verifytree.ReadMarker` and `shuttleengine.ReadWaitMarker`, composed in `internal/loomcli/reviewstatus.go`.
- Driver stencil: `## Notifying the parent` and `## Stopping on a non-running state` in `contracts/stencils/shed/shed-template-driver.md`; the park marker is written by the park command, its path declared in `internal/shedrun/paths.go`.
- Orch stencil: the start step in `contracts/stencils/orch/orch-template-role.md`.
- Reed: pane enumeration targets `=<session>:` today (`internal/reedengine/overlay.go`), as do the window-size, resize-hook, generation and attach-geometry reads (`exactSessionWindowTarget`), of which the generation read needs only the session; `remain-on-exit` is set globally at boot in `lifecycle.go`; reed state records `SelvagePaneID`, a pane id, and no window id; `TmuxCmd.listPanes` and `ListSessions` are engine-less, so the resolved target is passed to them; the required verbs are in `probe.go`.
- Driver liveness: `DriverAlive` in `internal/battencli/wire.go` is a reed status call, and the notice destination there is `orchcli.PrimePaths`.
- Awaiting without a `parent_notice`: the PR-Gate in `internal/landingshed/prgate.go` returns `Awaiting` with a reason only; the Bouncer in `internal/shedadapters/bouncer.go` sets the notice.
- Driver render: `driverPrompt` in `internal/loomcli/driverprompt.go` runs at a fresh launch only; `resumeParkedDriver` renders nothing.
