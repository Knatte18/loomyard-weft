# Decision record — orch-strand-cycle

## Goal

The orch lifecycle holds up when a session, its watcher or the whole machine goes away: dead strands read dead and free their names, a dead watcher comes back, and the context cycle needs no reply, can be withdrawn and never re-sends a delivered `/compact`.

## Scope

In:

- Reed's strand liveness judgment and its readers (board section 1).
- Reed never killing a recorded pane that sits outside the reed session, only naming it.
- Watcher restart at `lyx orch status` and at every `lyx orch start` (section 2).
- The note gate without a `written` reply; `defer` kept for a soft cycle (section 3).
- `lyx orch withdraw`, the pending request in `lyx orch status`, and the hub-pause notice on `distill` and `refresh` (section 4).
- Reclaiming a dead strand's name at a reed add (folded `reclaim-dead-strand-names`: issues #799, #802 and the name-clash part of #811).
- Issue #828: creel never re-typing a pasted inject, the watcher withdrawing a still-queued `/compact` once the compaction is seen, and creel's strand listing skipping a worktree torn down mid-lookup.
- Docs in the same commit: the `doc.go` of each package changed, the orch CLI help, and the orch handoff and role stencils.

Out:

- The `loop-exited` halt-note part of #811: task `loop-exit-halt-names-its-cause` owns it, so #811 closes only when both land.
- Removing the hub orch rules section "Dead strands after a reboot (#799)": the section says to remove it once #799 is fixed, and a task's code change never writes Bolt, so the orch removes it with `lyx orch rules` after landing; the PR body names that step.
- Adopting or moving a pane outside the reed session.
- A batten verb that ends a run: task `batten-cancel-verb` depends on this task and builds on D1.

## Decisions

### D1. One strand liveness judgment in reed

Decision: reed judges a strand live only when all of these hold:

1. Its recorded pane is listed and not dead, as today.
2. The pane sits in this worktree's reed session; today's listings are already scoped to it, and D2 names a pane found elsewhere.
3. The pane's foreground command, tmux's `#{pane_current_command}`, is not the shell reed launched the pane with, compared by base name with a leading `-` dropped.

Readers: reed `Status` (so `lyx orch status`, `lyx orch start`, webster's `StrandLive`, loom's driver check, creel's `Strand.Live` and shuttle's wait and attach), reed's directory rows, Resume planning, the `--if-absent` classification and D6's reclaim.
Pane-level checks keep conditions 1 and 2 only: the Selvage pane's health, pane-option reassertion and title repair act on the pane, and Selvage is a shell by design.

Consequences, stated so dependants see them:

- Resume relaunches a shell-only strand once its grace has passed, killing its old pane, so a resume brings back an engine the operator exited on purpose.
- Shuttle's stop removes a strand whose pane is present, in the shared window as in an own window: reconcile kills only a pane tmux reports dead, so a shell-only pane in the shared window would otherwise stay.

Departures from the operator decision of 2026-10-21, with their reason:

- Condition 3 tests "not the shell", not "is `claude`": reed holds no provider knowledge (`PATTERN-shuttle-provider-seam`), a Node-based or version-named engine binary can report another name, and a false "dead" makes `start` remove a live orch.
  A pane where the operator started another program after the engine exited reads live.
- Condition 2 checks the session, not the exact window: a pane the operator moved between windows of the reed session still runs reed's engine, and reading it dead would make `start` launch a second orch beside it.
- On Windows condition 3 is skipped: a PowerShell-native cmdlet runs inside the shell, so the shell's name is no evidence the engine exited.

Bound of condition 3:

- A launch grace of 30 seconds skips condition 3 for a strand whose latest launch is younger, since its shell is foreground until the sourced launch line starts the engine.
  Reed records the launch time on the strand at every launch, a new optional field; a strand with none was launched before this change and gets condition 3 with no grace.
- A backend that answers an empty foreground command skips condition 3.

Rationale: in #746 the engine exited and the pane fell back to the interactive shell its launch line was sourced in, so the pane stayed present and not dead.

### D2. A pane outside the reed session is never touched, only named

Decision: when a strand's recorded pane id is not listed in the reed session, reed asks the server once for that id across all sessions.
Found elsewhere, the strand is dead per D1, its status row names the placement (`session:window`), and reed's strand removal drops only the binding and never kills that pane.
`lyx orch status` carries it as `strand_elsewhere`, empty when none.
`start --continue` and `--adopt` launch orch in the prime's reed session as today.

Rationale: pane ids are server-global, so removal's `kill-pane` by id would end a pane in a session the operator made; relaunch already kills only a pane listed in the reed session.
Bound: the server-wide lookup runs only when a strand's pane is absent from the session.

### D3. A dead watcher is restarted at `status` and `start`

Decision: `lyx orch status` restarts the watcher when the orch strand is live per D1 and no watcher holds `watch.lock`, and reports `watcher_restarted: true`.
It takes `start.lock` without waiting and restarts nothing when it is held.
`lyx orch stop` takes `start.lock` for its strand removal, so a `status` restart never binds a watcher to a strand a concurrent stop removes.
`lyx orch start` keeps its branch that spawns a watcher for a live strand.
Both sites log `orch: restarted a dead watcher` at Info with the previous watcher's recorded exit.

Rationale: `status` is what the orch and operator run when the orch goes quiet.
Bound: `status` writes only this one spawn; two concurrent `status` calls may spawn two watchers, and `watch.lock` keeps one.
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

- `lyx orch withdraw` removes a pending cycle request of either mode and answers `withdrawn: true`.
  With none pending it answers `withdrawn: false` with the phase and `cycle_trigger`; it refuses only while a requested cycle runs (phase not idle, trigger `requested`), naming the phase, with the way forward to wait for the cycle to end and read `lyx orch status`.
  A request made during a hard or soft cycle is pending, and is withdrawn.
- Withdraw and the watcher's requested-cycle start serialize on the orch state lock.
  Withdraw checks and removes the marker under it.
  The watcher, inside the locked save that persists the requested cycle's first phase, enters only when the marker still holds the mode and request time it read, and removes it there; otherwise it releases the creel hold it took and stays idle.
  So a withdraw answering `withdrawn: true` started no cycle, and a request written between the read and the save is never consumed by a cycle of the old request.
- `lyx orch status` gains `cycle_request`: the pending request's mode, request time and `held_by_pause`, true while a live hub pause entry holds the watcher's cycle machine; null when none is pending.
- `distill` and `refresh` add `held_by_pause: true` and a `note` that the request waits for the hub resume, when a live hub pause entry holds the cycle machine; the request is still written.

Rationale: under a hub pause the orch deleted `.lyx/orch/cycle-request` by hand; the verb and fields replace that and say why a request waits.
Bound: withdraw removes only the request marker, never a started cycle's state.

### D6. A reed add reclaims a dead strand's name

Decision: before `AddStrand` forms a name, reed removes a strand holding the name the add asks for, the explicit name or the un-numbered role name, when that holder is not live per D1, not hidden (`anchor:hidden`) and the pair is not paused; numbering then runs over what remains.
The removal is reed's ordinary strand removal, with D2's guard, and logs `reed: reclaimed a dead strand's name` at Info with the old guid.
A live or hidden holder, or any holder while the pair is paused, keeps the name, and the add numbers or refuses as today, so seatengine's `checkName` still refuses when the holder is provably live.
The `--if-absent` path keeps relaunching a matched dead strand, and `AddStrandUnless` keeps returning a named strand.
Creel addresses receivers by strand name, so mail queued for the dead holder goes to the new strand of the same name, and its queued injects run under their TTL there.

Rationale: #799, #802 and #811 failed or drifted only because a dead registration held the name, and reed's add is the one site every respawning caller goes through.
Bound: only the asked-for name is reclaimed; a dead strand holding a numbered form stays.
Resume planning holds a paused pair's dormant strands on purpose, so reclaim leaves them alone.

### D7. A pasted inject is never typed again

Decision:

- Creel's delivery maps shuttle's post-paste failures, `ErrSentTextInDraft` and `ErrForeignBoxText`, onto `ErrNotLanded`, checked before the generic `ErrSessionBusy` that maps onto `ErrBusy`; only a hold before the paste stays `ErrBusy` and is requeued.
- The watcher's compacting phase, on finding a qualifying compaction boundary while its `/compact` inject has no outcome, withdraws its queued creel items before completing the phase, and clears the resume-noticed mark of a hub-pause resume line the withdraw removed, as the watcher's other withdraw sites do.
- Creel's `Strands` skips a listed worktree that no longer resolves as a worktree because it is being torn down: its directory is gone, or the resolve fails as not a git repository; it logs the skip at Info.
  Any other resolve or read failure still fails the call.

Rationale: creel's contract is that `ErrBusy` means nothing was typed, and both errors are raised after the paste.
Evidence: the creel journal of 2026-10-22 shows inject 121, the `/compact`, with `sending` at 12:21:08, 12:21:37 and 12:22:49, each followed by a `draft` stall and none by an outcome, which is the requeue of a busy answer, not an expiry.
`watch.log` shows `creel: resolve worktree …/.fabric-wt-staging-…/decision-add-notifies: not a git repository`, a worktree mid-teardown.
Bound: an inject left beside a draft is not retyped; shuttle's `pending-send.json` resolves an `ErrSentTextInDraft` one at the next send, and an `ErrForeignBoxText` or recordless one is lost rather than typed twice.

## Constraints

- `PATTERN-shuttle-provider-seam`: reed names no engine; D1 compares against reed's own shell.
- `PATTERN-tolerant-records`: the strand's launch time is a new optional `reed.json` key written through the existing state save.
- `PATTERN-refusal-way-forward`: D5's refusal names its way forward, and a test reaches it.
- `PATTERN-run-process-owner`: the orch watcher is a hub process outside any run and stays outside `runproc`, as today; D3 adds a spawn site of the existing path.
- Fact (tmux manual, 3.x; tmux 3.4 on the Linux host of #746): `#{pane_current_command}` is the name of the pane's foreground process group leader, read through the pane's tty.

## Auto-mode assumptions

- D1: grace 30 seconds, a constant; "not the shell"; session-level placement; Windows skips condition 3.
- D3: `status` restarts only for a live strand.
- D4: a `written` reply is ignored.
- D5: verb name `withdraw`.
- D6: only the asked-for name, never during a pair pause.

## Open risks

- An engine that makes a child shell the tty's foreground process group would read dead after the grace; Claude Code's tool commands stay in its own process group today.
- A strand whose command is itself a shell, as a test helper may add, reads dead once the grace passes.
- A composite launch command (`a; b`) has a moment between its parts with the shell in the foreground.
- Condition 3 is unverified on Windows, where it is skipped.

## Acceptance criteria

- A strand whose engine exited and whose pane holds only its shell reads `live: false` once the grace has passed, and `live: true` within it; one recorded before this change reads dead at once.
- The Selvage pane is never read dead or replaced for running its shell.
- `lyx orch status` shows `strand_live: false` for such an orch strand, and `lyx orch start --continue` resumes it in one step.
- A strand whose recorded pane is found only in another session reads dead, names the placement, shows `strand_elsewhere` in `lyx orch status`, and reed's removal does not kill that pane.
- Webster's `StrandLive`, loom's driver check and creel's `Strand.Live` report a shell-only strand as not live; shuttle's stop removes a shell-only strand in the shared window.
- `lyx orch status` with a live strand and no watcher restarts it, reports `watcher_restarted: true` and logs it; with `start.lock` held it restarts nothing.
- The note phase moves on with no reply once a turn end is read and the note exists non-empty; an empty or missing note keeps waiting; a restarted watcher finds the turn end from the phase's events offset.
- A `defer` to a soft request with the note unwritten still returns the watcher to idle with `deferred`.
- `lyx orch withdraw` removes a pending request, answers `withdrawn: false` with none pending, and refuses with its way forward while a requested cycle runs.
- A withdraw racing the requested-cycle start either withdraws with no cycle started or is refused because it started; a refresh written after a withdraw in that race is not consumed by the withdrawn request's cycle, and the refused entry releases its creel hold.
- `lyx orch status` shows a pending request with its mode, time and `held_by_pause`; `distill` and `refresh` say so under a hub pause.
- A reed add asking for a name a dead, non-hidden strand holds gets that exact name, by role as by explicit name, and the dead strand leaves `lyx reed list`; a live holder still numbers the name and seatengine still refuses.
- A row respawning after every pane died (the #799 and #811 shape) starts under its canonical name and does not fail.
- A pasted send failing with `ErrSentTextInDraft` or `ErrForeignBoxText` is journaled unconfirmed and not typed again; a hold before the paste is requeued.
- A compaction boundary found while the `/compact` inject is still queued withdraws it.
- Creel's strand listing skips a worktree whose directory is gone or no longer a git worktree, and still fails on any other unreadable worktree.
