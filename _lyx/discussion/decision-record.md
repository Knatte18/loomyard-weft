# Decision record — orch-strand-cycle

## Goal

The orch lifecycle holds up when a session, its watcher or the whole machine goes away: dead strands read dead and free their names, a dead watcher comes back, and the context cycle needs no reply, can be withdrawn and never re-sends a delivered `/compact`.

## Scope

In:

- Reed's strand liveness judgment and its readers (board section 1).
- Reed naming a recorded pane that sits outside the reed session, which its removal already never kills.
- Watcher restart at `lyx orch status` and at every `lyx orch start` (section 2).
- The note gate without a `written` reply; `defer` kept for a soft cycle (section 3).
- `lyx orch withdraw`, the pending request in `lyx orch status`, and the hub-pause notice on `distill` and `refresh` (section 4).
- Reclaiming a dead strand's name at a reed add (folded `reclaim-dead-strand-names`: issues #799, #802 and the name-clash part of #811).
- Issue #828: creel never re-typing a pasted inject, the watcher withdrawing a still-queued `/compact` once the compaction is seen, and creel's strand listing skipping a worktree torn down mid-lookup.
- Docs in the same commit: the `doc.go` of each package changed, the orch CLI help, the orch handoff and role stencils, and the claude.yaml `binary` key's template comment.

Out:

- The `loop-exited` halt-note part of #811: task `loop-exit-halt-names-its-cause` owns it, so #811 closes only when both land.
- Removing the hub orch rules section "Dead strands after a reboot (#799)": the section says to remove it once #799 is fixed, and a task's code change never writes Bolt, so the orch removes it with `lyx orch rules` after landing; the PR body names that step.
- Adopting or moving a pane outside the reed session.
- A batten verb that ends a run: task `batten-cancel-verb` depends on this task and builds on D1.

## Decisions

### D1. One strand liveness judgment in reed

Decision: reed judges a strand live only when all of these hold:

1. Its recorded pane is listed and not dead, as today.
2. The pane sits in this worktree's reed session.
   Today's code meets this for every strand that owns a pane: `validateDisplay` admits only the `own-window`, `overview` and `hidden` anchors (internal/reedengine/strand.go), the retired `below-parent` and `top` decode as `own-window`, and a hidden strand owns no pane, so every bound pane is `isOwnWindow`.
   `presentPanes` therefore already reads every bound pane from the session-wide `listWindowPanes` listing (internal/reedengine/ownwindow.go), and reconcile already keeps the binding of a pane that listing holds, so a pane the operator moved to another window of the session reads present with no listing, reconcile or render change.
   D2 names a pane found outside the session.
   Condition 3 is D1's one listing change: `presentPanes` takes a bound pane from the strands'-window listing (`paneListFormat`, internal/reedengine/overlay.go) and adds from the session-wide listing (`windowPaneFormat` and `windowPane.livePane`) only a pane the first lacks, and neither format carries `#{pane_current_command}` or `#{pane_start_command}`, so both fields join both formats and the `LivePane` they map to.
   A pane read without the fields would have an empty start command and skip condition 3, so a format left without them silently keeps the #746 false-live.
   `#{pane_start_command}` can hold spaces, as the pane title, today the space-bearing last field both parsers split off with `SplitN`, does, so the two new fields need a delimiter or a field order the parsers can split.
3. The pane's foreground command, tmux's `#{pane_current_command}`, is not the shell the pane was started with, the pane's own `#{pane_start_command}`, compared by base name with a leading `-` dropped.
   A strand pane splits with no command, so tmux freezes the session's pinned `default-command`, the shell, into that pane at spawn, when the session holds that pin.
   The pin is session scoped and each of its `set-option` calls is non-fatal (`shellOptionArgvs` and `pinGeometryOptionsLocked`, internal/reedengine/windowsize.go): an empty shell path or a failed `set-option` leaves the session unpinned, and a session booted before the pin existed has none until the attach pre-flight pins it, so a pane it spawned before that keeps an empty start command.
   On tmux 3.6, a pane split with no command in a session with `default-shell` set and no `default-command` lists `#{pane_start_command}` as empty.
   A pane whose `#{pane_start_command}` is empty skips condition 3, so it reads live, as a pane with an empty foreground command does (Bound below).
   The per-pane value, not the session pin or the `shell` config key, is the comparand, because the attach pre-flight re-pins the session to the currently resolved shell, so a changed key would otherwise misjudge a pane the old pin started.

Readers: reed `Status` (so `lyx orch status`, `lyx orch start`, webster's `StrandLive`, loom's driver check, creel's `Strand.Live` and shuttle's wait and attach), reed's directory rows, Resume planning, the `--if-absent` classification and D6's reclaim.
Pane-level checks keep conditions 1 and 2 only: the Selvage pane's health, pane-option reassertion and title repair act on the pane, and Selvage is a shell by design.

Consequences, stated so dependants see them, where a shell-only strand or pane is one whose pane holds only its shell and lists a non-empty `#{pane_start_command}`, since one with an empty start command reads live:

- Resume relaunches a shell-only strand once its grace has passed, killing its old pane, so a resume brings back an engine the operator exited on purpose.
- Shuttle's stop removes a shell-only strand as today: `strandRemovable` (internal/shuttleengine/stop.go) takes an own-window strand whose pane is present whatever its liveness, so D1 changes nothing there.
- Landing's conflict-session stop (`conflictSessionStopper`, internal/loomcli/landingdeps.go) removes a present shell-only conflict strand, as shuttle's stop does; today it skips a strand that is not live, so under this judgment it would leave that strand's pane and record.
- Every removal that reads this judgment kills a present pane that holds a bare shell: D6's reclaim, `start`'s `StartRelaunch` `RemoveStrand`, Resume's relaunch through `killReplacedOwnWindowPaneLocked` and landing's conflict-session stop.
  A pane misread as dead under the residual cases of the bound below is killed with its engine.

Departures from the operator decision of 2026-10-21, with their reason:

- Condition 3 tests "not the shell", not "is `claude`": reed holds no provider knowledge (`PATTERN-shuttle-provider-seam`); support-log.md's Rejected alternatives holds the engine-name alternative.
  A pane where the operator started another program after the engine exited reads live.
- Condition 2 checks the session, not the exact window: a pane the operator moved between windows of the reed session still runs reed's engine, and reading it dead would make `start` launch a second orch beside it.
- On Windows condition 3 is skipped: a PowerShell-native cmdlet runs inside the shell, so the shell's name is no evidence the engine exited.

Bound of condition 3:

- A launch grace of 30 seconds skips condition 3 for a strand whose pane is younger, since its shell is foreground until the sourced launch line starts the engine.
  The pane's age is the elapsed time of its first process, the `#{pane_pid}` reed's listings already read: the system uptime from `/proc/uptime` less that process's start time, field 22 of `/proc/<pid>/stat` in clock ticks since boot, a tick being the kernel's `USER_HZ` that `sysconf(_SC_CLK_TCK)` returns, read through cgo, which the `lyx` build already needs; both are read as reed's Linux process probes read `/proc` (internal/reedengine/proctree_linux.go), so a `Status` read spawns no process.
  An age that cannot be read counts as inside the grace, so condition 3 is skipped for that read and a failed read never makes a removal edge kill a pane.
  Every launch makes a fresh pane and reed never respawns one, so that age is the latest launch's age whichever `lyx` binary launched it.
  No launch time goes on the strand: the pane prelude pins `lyx` to the binary that spawned the pane, so old-binary runs keep adding strands after this change, and internal/state drops unknown keys from a slice element updated in place (internal/state/doc.go), so an old binary's update would strip such a key.
- Misjudged live: tmux reports the foreground process group leader's name, so a live engine reads as the shell, and dead once the grace passes, in two cases.
  An engine started through a shell wrapper that does not `exec` it reads as the wrapper's shell; an engine suspended with Ctrl-Z leaves the shell as the foreground group.
  The wrapper case is bounded by a Constraint: the engine binary must be the engine or `exec` it.
  The suspended case is a residual: a suspended engine reads dead after the grace and every removal edge named in the Consequences kills it.
- On tmux a failed foreground read answers the shell (Constraints, the tmux fact), so the residual is a read that races the foreground group's exit, which then reads dead.
  A backend that answers an empty foreground command skips condition 3.
- A pane whose `#{pane_start_command}` is empty, because its session held no `default-command` pin at its spawn, skips condition 3 and reads live, so the #746 false-live stays for that pane.

Rationale: in #746 the engine exited and the pane fell back to the interactive shell its launch line was sourced in, so the pane stayed present and not dead.

### D2. A pane outside the reed session is never touched, only named

Decision: when a strand's recorded pane id is not listed in the reed session's session-wide listing (D1 condition 2), reed asks the server once for that id across all sessions.
Found in any session other than the reed session, the pane is elsewhere: the strand is dead per D1, and its status row names the placement (`session:window`).
`lyx orch status` carries it as `strand_elsewhere`, empty when none.
Reconcile keeps the binding of a strand whose pane the same lookup finds in another session, where today `planReconcile` clears every own-window binding the session listing lacks (internal/reedengine/reconcile.go), so the placement survives the reconcile every public reed op but `Status` runs first, and stays on the status row until the pane leaves the server or a removal drops the binding.
`lyx orch start` is such a removal for the orch strand: the strand reads dead, so start's `StartRelaunch` removes it and launches a new one, and the placement shows in `lyx orch status` until then.
`start --continue` and `--adopt` launch orch in the prime's reed session as today.
Removal keeps today's guard and drops only the binding of such a strand.

Rationale: today's removal already kills only panes the reed session lists: `killStrandPanes` narrows its kill list with `paneIDsInSession` over the session's listing (internal/reedengine/strand.go), and `killReplacedOwnWindowPaneLocked` kills only a pane the session lists.
New here are the server-wide lookup, the kept binding, the placement on the status row and `strand_elsewhere`, so the operator learns where a pane they moved out went.
The #746 observation, `%3 0:bash bash`, does not establish such a pane: it came from a `tmux list-panes` whose server the issue does not name, pane ids are per server, reed runs its own `-L` socket, and `0` is the default server's first session name; D2 covers the case a pane reaches another session of reed's server, which nothing seen so far rules out.
Bound: the server-wide lookup runs only when a strand's pane is absent from the session; a pane in another tmux server is not found and its strand's binding is cleared as today.
Reed's server is shared by the hub's worktrees and numbers pane ids afresh when it restarts, so after a reboot a stale binding can match a sibling worktree's new pane of the same id: the strand still reads dead, as it is, but its row names the sibling's placement until a removal drops the binding or that pane goes.

### D3. A dead watcher is restarted at `status` and `start`

Decision: `lyx orch status` restarts the watcher when the orch strand is live per D1 and no watcher holds `watch.lock`, and reports `watcher_restarted: true`.
It takes `start.lock` without waiting and restarts nothing when it is held.
`lyx orch stop` takes `start.lock` for its strand removal, so a `status` restart never binds a watcher to a strand a concurrent stop removes.
`lyx orch start` keeps its branch that spawns a watcher for a live strand.
Both sites log `orch: restarted a dead watcher` at Info with the previous watcher's recorded exit.

Rationale: `status` is what the orch and operator run when the orch goes quiet.
Bound: `status` writes only this one spawn; two `status` calls in sequence, the second before the first one's watcher takes `watch.lock`, may spawn two watchers, and `watch.lock` keeps one.
A dead strand gets no watcher from `status`: `start` relaunches strand and watcher together.

### D4. The note gate needs no reply

Decision: in phase handoff-requested the watcher ends the note phase at a tick where the newest session event read since the phase began is a turn end and the requested note file exists and is not empty.
A restarted watcher re-reads from the phase's persisted events offset, so it finds a turn end read before the restart.
The handoff and role stencils stop asking for a `written` reply, so the note request is a post that expects no answer except a soft cycle's optional `defer`.
A `defer` to a soft request still declines the cycle while the note is unwritten; one read in the same tick as a passing gate is ignored.
A `written` reply from an older role file is logged and ignored.
The guarantee becomes: nothing reaches `/clear` or `/compact` before the named note exists, non-empty, at a turn end of the note phase.

Rationale: the watcher already reads turn ends, and the note path is new per cycle.
Bound: a note the orch extends in a later turn is cycled on the version present at the first qualifying turn end.

### D5. `lyx orch withdraw`, the visible request and the pause notice

Decision:

- `lyx orch withdraw` removes a pending cycle request of either mode, in any phase, and answers `withdrawn: true`.
  A cycle start clears the marker it consumes (`tickIdle` in internal/orchengine/watcher.go), so a marker present during a cycle has not started, and a request made during any cycle, a requested one included, is withdrawn.
  With no marker pending it answers `withdrawn: false` with the phase, and with `cycle_trigger` only outside the idle and resuming phases, so a caller whose request already started learns which cycle runs.
  It never refuses, and never reads `CycleTrigger` in those two phases: `toIdle` does not reset that field and `startReload` enters the resuming phase without setting it, so there it can name an earlier cycle.
  This departs from the board entry, which asks the verb to refuse once the cycle has started: the answer already names the running cycle's phase and trigger, and a refusal would make a caller that lost a race it cannot avoid, the cycle start consuming its request, read as a failed command.
- Every watcher removal of the marker serializes with withdraw on the orch state lock, as one compare-and-remove of the mode and request time the watcher read: the cycle start of every trigger, the stale-request removal and `removeSatisfiedRequest` (internal/orchengine/watcher.go).
  Withdraw checks and removes the marker under the same lock.
  `RequestCycle` (internal/orchengine/state.go) writes the marker under that lock by a temp-file rename, so a locked reader never sees a truncated marker, and `requestCycle`'s removal when no watcher is live (internal/orchcli/refresh.go) is a compare-and-remove of the request it just wrote under the same lock, so every writer and remover of `cycle-request` holds the state lock.
  A cycle start compares and removes inside the locked save that persists the cycle's first phase.
  A hard or soft cycle start consumes a request pending at its read, as today.
  When the marker no longer holds what the watcher read, a requested start releases the creel hold it took and stays idle, a hard or soft start enters and leaves the newer marker pending for the next idle, and the other two removals leave it.
  So a withdraw answering `withdrawn: true` started no cycle, and a request written after a removal's read is never removed by it.
- The stale-request removal measures a request's age excluding the time a hub pause entry covered it, read from the pause record's hub intervals (`pauserecord.Record.CoverageOf` with an empty slug), so a request held by a pause waits for the resume and expires only after `handoff_timeout_s` of unpaused time.
  The record keeps a resumed entry for `pauserecord.Retention`, so a pause resumed longer ago than that no longer discounts the age.
- The handoff phase's timeout of a requested cycle excludes the same paused time: `tickHandoff` starts that clock at `CycleRequestedAt` when it predates `PhaseEnteredAt`, and measures the elapsed time from that start less the hub pause coverage read from the same `CoverageOf` intervals, so a request a pause held runs its cycle after the resume instead of timing out on the first handoff tick.
- `lyx orch status` gains `cycle_request`: the pending request's mode, request time and `held_by_pause`, true while a live hub pause entry holds the watcher's cycle machine; null when none is pending.
- `distill` and `refresh` add `held_by_pause: true` and a `note` that the request waits for the hub resume, when a live hub pause entry holds the cycle machine and a live watcher kept the request; the request is still written.
  With no live watcher the request is removed and the verbs answer as today.

Rationale: under a hub pause the orch deleted `.lyx/orch/cycle-request` by hand; the verb and fields replace that and say why a request waits.
Bound: withdraw removes only the request marker, never a started cycle's state.

### D6. A reed add reclaims a dead strand's name

Decision: before `AddStrand` forms a name, reed removes a strand holding the name the add asks for, the explicit name or the un-numbered role name, when that holder is not live per D1, not hidden (`anchor:hidden`) and the pair is not paused; numbering then runs over what remains.
The removal is reed's ordinary strand removal, with removal's existing session-membership guard, and logs `reed: reclaimed a dead strand's name` at Info with the old guid.
A live or hidden holder, or any holder while the pair is paused, keeps the name, and the add numbers or refuses as today, so seatengine's `checkName` still refuses when the holder is provably live.
The `--if-absent` path keeps relaunching a matched dead strand, and `AddStrandUnless` keeps returning a named strand.
Creel addresses receivers by strand name, so mail queued for the dead holder goes to the new strand of the same name, and its queued injects run under their TTL there.

Rationale: #799, #802 and #811 failed or drifted only because a dead registration held the name, and reed's add is the one site every respawning caller goes through.
The #811 holder was dead, read from the code, since the issue's notes hold no reed state: a burler fix attempt runs as a table's chair (internal/burlerengine/fix.go), which keeps no pane (`seatSpec`, internal/seatengine/prompt.go), and every end of a started chair removes its strand: a `done` run through shuttle's finalize (internal/shuttleengine/wait.go), a timeout or wait error through `stopSeat` in seatengine's `join` (internal/seatengine/engine.go), and a `died` own-window run through its died-window removal.
So whichever way the stuck attempt's run ended, its strand was removed unless that removal was skipped or failed: `removeStrandIfPresent` skips a strand that is neither live nor present (internal/shuttleengine/stop.go), the registration of a pane that already went; a failed removal in the finalize or died-window path logs a Warn and goes on, and a failed `stopSeat` fails the attempt with a `lyx reed remove` way forward.
The skipped case leaves the dead registration D6 reclaims.
A failed removal leaves either a live pane, whose retry `checkName` still refuses, which stays right while that pane may still write, or a dead registration D6 reclaims; D6 adds no teardown for it.
Bound: only the asked-for name is reclaimed; a dead strand holding a numbered form stays.
A holder inside its D1 launch grace reads live and keeps the name, so a retry within 30 seconds of the holder's pane launch is numbered and refused by seatengine as today.
The reclaim kills a holder's present shell-only pane, in D1's Consequences' sense, and a holder misread as dead under D1's residual cases is killed with its engine.
Resume planning holds a paused pair's dormant strands on purpose, so reclaim leaves them alone.

### D7. A pasted inject is never typed again

Decision:

- Creel's delivery maps shuttle's post-paste failures, `ErrSentTextInDraft` and `ErrForeignBoxText`, onto `ErrNotLanded`, checked before the generic `ErrSessionBusy` that maps onto `ErrBusy`; only a hold before the paste stays `ErrBusy` and is requeued.
- The watcher's compacting phase, on finding a qualifying compaction boundary while its `/compact` inject has no outcome, withdraws its queued creel items before completing the phase, and clears the resume-noticed mark of a hub-pause resume line the withdraw removed, as the watcher's other withdraw sites do.
- Creel's `Strands` skips a listed worktree that no longer resolves as a worktree: its directory is gone, or the resolve fails as not a git repository.
  A not-a-git-repository resolve cannot tell a teardown from a broken `.git` link, so any worktree whose resolve answers it is skipped whatever the cause, and its strands get no delivery until it resolves again.
  The skip logs at Info when the worktree's directory or the gitdir its `.git` link names is gone, a teardown's mark, and at Warn when both still exist, so a cause that is no teardown shows.
  It also skips a worktree whose reed state read, wire or reed status step fails when its directory, checked after that failure, no longer exists, since a teardown can land after a successful resolve.
  Any other resolve or read failure still fails the call.

Rationale: creel's contract is that `ErrBusy` means nothing was typed, and both errors are raised after the paste.
Evidence: the creel journal of 2026-10-22 shows inject 121, the `/compact`, with `sending` at 12:21:08, 12:21:37 and 12:22:49, each followed by a `draft` stall and none by an outcome, which is the requeue of a busy answer, not an expiry.
`watch.log` shows `creel: resolve worktree …/.fabric-wt-staging-…/decision-add-notifies: not a git repository`, a worktree mid-teardown.
Bound: an inject left beside a draft is not retyped; shuttle's `pending-send.json` resolves an `ErrSentTextInDraft` one at the next send, and an `ErrForeignBoxText` or recordless one is lost rather than typed twice.

## Constraints

- `PATTERN-shuttle-provider-seam`: reed names no engine; D1 compares against reed's own shell.
- `PATTERN-tolerant-records`: D1 adds no `reed.json` key; the launch grace reads the pane's own process age.
- The engine binary, the claude.yaml `binary` key (internal/shuttleengine/claudeengine/template.yaml), must be the engine or `exec` it, and that key's template comment says so; D1's condition 3 reads a non-exec wrapper's shell as the engine having exited.
- `PATTERN-run-process-owner`: the orch watcher is a hub process outside any run and stays outside `runproc`, as today; D3 adds a spawn site of the existing path.
- Fact (tmux manual, 3.x; tmux 3.4 on the Linux host of #746, 3.6 on this host): `#{pane_current_command}` is the name of the pane's foreground process group leader, read through the pane's tty.
  When that read fails, tmux 3.4 and 3.6 answer the pane's argv, which reed's pinned `default-command` makes the shell, and with no argv the pane's shell (`format.c` `format_cb_current_command`); on Linux the read fails when `tcgetpgrp` fails or the leader's `/proc/<pgrp>/cmdline` is unreadable or empty (`osdep-linux.c` `osdep_get_name`).

## Auto-mode assumptions

- D1: grace 30 seconds, a constant; "not the shell"; session-level placement; Windows skips condition 3.
- D3: `status` restarts only for a live strand.
- D4: a `written` reply is ignored.
- D5: verb name `withdraw`; it never refuses, departing from the board entry's refusal once the cycle has started.
- D6: only the asked-for name, never during a pair pause.

## Open risks

- An engine that makes a child shell the tty's foreground process group would read dead after the grace.
  Claude Code's tool commands stay off the tty today: on Claude Code 2.1.308 under tmux 3.6, `ps -t` over an agent pane's tty during a running tool command lists only the pane's bash and claude, with claude as the tty's foreground process group.
- An rc file that `exec`s another shell leaves that shell in the foreground after the engine exits, so the strand reads live: a false-live residual.
- A strand whose command is itself a shell, as a test helper may add, reads dead once the grace passes.
- A composite launch command (`a; b`) has a moment between its parts with the shell in the foreground.
- Condition 3 is unverified on Windows, where it is skipped.
- A shell-only pane spawned in a session without the `default-command` pin has an empty start command, so it keeps the #746 false-live.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - empty-start-command-no-outcome (class design, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - card5-pane-lookup-warn-every-poll (class design, severity BLOCKING, open in rounds 2)
  - card7-skip-line-repeats-per-call (class design, severity BLOCKING, open in rounds 2)
  - card7-worktreetorndown-per-symbol-test (class design, severity BLOCKING, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- On Linux, a strand whose engine exited and whose pane holds only its shell and lists a non-empty start command reads `live: false` once its pane is older than the grace, and `live: true` within it, whichever `lyx` binary launched it; on Windows such a strand reads live, since condition 3 is skipped there.
- A shell-only pane whose `#{pane_start_command}` is empty reads live.
- An own-window strand whose pane the operator moved to another window of the reed session reads live and is not relaunched.
- The Selvage pane is never read dead or replaced for running its shell.
- `lyx orch status` shows `strand_live: false` for such an orch strand, and `lyx orch start --continue` resumes it in one step.
- A strand whose recorded pane is found only in another session of reed's server reads dead, names the placement, shows `strand_elsewhere` in `lyx orch status` also after a reconciling reed op that does not remove it, such as another strand's add, and reed's removal does not kill that pane.
- On Linux, webster's `StrandLive`, loom's driver check and creel's `Strand.Live` report a shell-only strand with a non-empty start command as not live; shuttle's stop removes a shell-only strand, and landing's conflict-session stop removes a shell-only conflict strand.
- `lyx orch status` with a live strand and no watcher restarts it, reports `watcher_restarted: true` and logs it; with `start.lock` held it restarts nothing.
  `lyx orch stop` takes `start.lock`, so a `status` restart never binds a watcher to a strand a concurrent stop removes.
- The note phase moves on with no reply once a turn end is read and the note exists non-empty; an empty or missing note keeps waiting; a restarted watcher finds the turn end from the phase's events offset.
- A `defer` to a soft request with the note unwritten still returns the watcher to idle with `deferred`.
- `lyx orch withdraw` removes a pending request and answers `withdrawn: false` with the phase when none is pending, with `cycle_trigger` outside the idle and resuming phases; a refresh written during a running requested cycle is withdrawn, and a withdraw during an auto-compaction reload after an earlier requested cycle is not refused.
- A withdraw racing the requested-cycle start either withdraws with no cycle started or answers `withdrawn: false` because it started; a refresh written after a withdraw in that race is not consumed by the withdrawn request's cycle, and the start that found the marker changed releases its creel hold.
- A refresh written between a hard start's read of the marker and its removal is not removed by that start: `removeSatisfiedRequest` removes it as satisfied when its mode is the cycle's, and otherwise it stays pending and runs at the next idle.
- `lyx orch status` shows a pending request with its mode, time and `held_by_pause`; `distill` and `refresh` say so under a hub pause.
- A request made under a hub pause longer than `handoff_timeout_s` is not removed as stale and runs its cycle to completion after the resume.
- A reed add asking for a name a dead, non-hidden strand holds gets that exact name, by role as by explicit name, and the dead strand leaves `lyx reed list`; a live holder still numbers the name and seatengine still refuses.
  A hidden holder, a holder inside D1's launch grace and any holder while the pair is paused keep the name, and the add numbers a role name or refuses an explicit name as today.
- A row respawning after every pane died (the #799 and #811 shape) starts under its canonical name and does not fail.
- A pasted send failing with `ErrSentTextInDraft` or `ErrForeignBoxText` is journaled unconfirmed and not typed again; a hold before the paste is requeued.
- A compaction boundary found while the `/compact` inject is still queued withdraws it, and the withdraw clears the resume-noticed mark of a hub-pause resume line it removed.
- Creel's strand listing skips a worktree whose directory is gone or no longer a git worktree, also when its directory goes after a successful resolve and its reed state read, wire or status step fails, and still fails on any other unreadable worktree.
