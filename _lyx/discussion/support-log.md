# Support log — lifecycle-waits

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry as it stood at the start (orch compact reload, step watchdog, issue #433) and the touching entries: `run-recovery` (done; built the wait markers and the verify timeout), `run-pushes` (done), `bugfix-fabric` (done), `parent-review-hardening` and `orch-owns-the-loop` (notes).
2. Compaction: read `tickCompacting` and `tick()` in `internal/orchengine/watcher.go` and the Claude transcript readers in `internal/shuttleengine/claudeengine/usage.go`.
   The board's diagnosis says the turn ends after compaction are not read; the transcript readers reopen the file at read time, so the stale turn end alone would not hide the boundary.
   Checked the hub orch transcript of 2026-10-07: the `compact_boundary` at 10:06:41Z is in the same file and is followed at 10:06:42Z by a queued operator message and its answer.
   `ContextTokens` returns only the newest qualifying entry, so that assistant entry hid the boundary.
   Fix: use `CompactedSince` with `PhaseEnteredAt` (D1).
   A first draft also stored every turn end read during compacting as the reading; `CompactedSince` uses the held turn end only for its transcript path, so that was dropped in burler round 1.
3. The stale `distill` side effect: `tickIdle` removes a request older than the handoff timeout; a request made during a ten-minute stalled cycle aged out.
   With D1 the cycle finishes in about a minute, so the request would instead trigger a second compaction; picked D2.
   `PhaseEnteredAt` is overwritten by every phase entry, so D2 removes the request when the cycle's action took effect, not by comparing with a phase time.
   The cycle's entry already clears a request pending at its start, so the later request needs no start time to be told apart (burler round 2 dropped the field).
4. First draft also designed the step watchdog, the Windows verify tree kill, and #433's push retry and rollback.
   The parent review (round 1) reported that the operator re-scoped the entry during Discussion to orch and batten only: those parts moved to the note `verify-watchdog` and the task `fabric-pushes`, and two sections were added: one notice channel to the orch, and a window per batten. Re-read the entry with `--body`.
5. Notice channel: read `internal/battenshed/innerrun.go`, `notice.go`, `batten-recipe.yaml`, the notice queue in `internal/orchengine/notice.go` and its watcher delivery, and the driver stencil's `## Notifying the parent`.
   Run-Shed bounces every 30 s by design (self-route with 1440 counted bounces); every bounce writes a history entry, folded only when identical and exempt.
   Picked a wait inside the call that returns only on a child state change, an arm event or a pause request, with a 2 s in-call check (D3).
6. Notice content: the stop report exists only after the driver writes it and parks (the park marker holds its path), so the state-changed notice waits for it up to 3 minutes; every notice gains `since` and `history` (D4).
   `since` is the status file's modification time at the check that saw the state, not the child's newest history entry: a pause or a producer failure appends no history entry (burler round 3).
7. Quiet: the status file moves only at step ends. Checked a 2026-10 transcript for an API-error turn: a synthetic main-chain assistant entry with `isApiErrorMessage: true`, `error: server_error` and the error text.
   Picked an activity reading in shuttle (transcript and events of live runs), a quiet condition on agent idleness with live wait markers excluded, and an `api-error` condition after 2 minutes idle (D5).
8. Driver messages: the driver messages the parent at every escalation and on friction failure; with batten owning state changes, a batten-watched driver messages only a `parent_notice` relay or a question (D6).
9. Window per batten: the orch role stencil starts battens with `setsid nohup`; reed owns tmux and the pane-binary chokepoint. Picked `lyx batten run <slug> --window` through a new reed op (D7).

## Rejected alternatives

- Compaction: re-reading `ContextTokens` through each turn end read during compacting, without `CompactedSince`. Any assistant entry after the boundary still hides it, which is the observed failure.
- Compaction: a transcript poll for any boundary newer than the phase entry, separate from `CompactedSince`. `CompactedSince` already does this with the transcript path from the held turn end.
- Compaction: storing the turn ends read during compacting as the reading. `CompactedSince` needs only the held turn end's transcript path, and D1 re-reads `ContextTokens` at completion.
- D2: leaving a same-mode request in place. It would start a second compaction a minute after the first.
- D2: dropping every request made during a cycle. A clear requested during a compact cycle asks for something the cycle did not do.
- D2: comparing `RequestedAt` with `PhaseEnteredAt`. Every phase entry overwrites it, so a request made during the handoff is kept and one made during the reload is removed.
- D3: keeping the bounce loop and only shortening `poll_interval_s`. History would grow faster, the opposite of what the operator asked.
- D3: a file-notification library (fsnotify) for the child's status file. A new dependency with platform quirks for a gain of under 2 s.
- D3: keeping a wall-clock limit on the running watch. A long run has no natural limit, and a limit hit ends nothing useful; the quiet notice reaches the orch instead.
- D4: sending the state-changed notice at once without the report path. The board asks that the notice carry the report, which is written after the stop.
- D4: superseding older queued notices for the same slug in the orch's queue. `since` and `history` already make a stale notice recognisable, and changing the queue's contract is not asked for.
- D4: a state-changed notice for an awaiting child beside the driver's `parent_notice`. The orch would get two messages for one event, and the relay carries the hand-off text only the driver holds.
- D4: no batten notice for any awaiting child. The PR-Gate hands off with no `parent_notice`, so the orch would never hear of a pull request waiting for its decision.
- D4: no batten notice, ever, for an awaiting child with a `parent_notice`. A dead driver or a failed relay send would then announce the stop by no channel; batten steps in when the driver strand has ended, no report came within 3 minutes, or the child is still undecided 10 minutes after the report.
- D4: leaving a failed or skipped relay from a live, reported driver as an accepted hole, bounded only by a failed send being written in the report. No notice points at that report, so nobody would read it; the 10-minute fallback bounds it instead, at the cost of one extra notice when the relay did succeed.
- D4: suppressing batten's awaiting notice for every batten. A batten outside the prime sends to the prime's orch while its driver relays to another worktree's orch, so only a batten that holds the watched marker suppresses.
- D4: taking `since` from the child's newest history entry. A pause or a producer failure appends no entry, so the time predates the change, the 3 minutes would already have passed and an earlier stop's report would qualify.
- D4: retrying a not-queued notice at every `notice_probe_s` without end. A stop with no orch strand would retry and log forever; batten does not call `QueueNotice` without a strand and retries a failed queueing 3 times.
- D3: driver liveness on every 2 s check. It is a reed status call (a multiplexer round trip), so it follows `notice_probe_s` like the other costly reads.
- D6: writing the watched marker for a batten started from any worktree. Its notices go to the prime's orch while its driver's parent is that worktree's orch, so only a batten in the prime silences its driver.
- D7: chaining `set-option remain-on-exit off` after a detached `new-window` without a target. Run against tmux 3.6, it lands on the session's current window, the strands' window; it targets the new window by its exact name.
- D7: relying on the chained option to fail when two calls share a window name. Run against tmux 3.6, a duplicate exact name does not fail the option: it lands on the older window and the new one keeps `remain-on-exit on`, so a batten that exits on the lock would leave a dead window. The op reads the option back by id and kills a window that does not read `off`.
- D7: falling back to the current window whenever no Selvage pane is live. A present Selvage corpse or a present strand pane names the strand window as well, so the current window is the last resort.
- D7: routing the pane-generation read through the strand-window seam. The read needs only session fields and is what validates the recorded pane ids the seam relies on, so it stays session-scoped.
- D7: resolving a recorded pane id's window through a server-wide pane target. Pane ids are server-wide, so a stale id could name another session's window; the seam reads the session's own pane listing.
- D6: writing the watched marker for any prime batten. The driver's parent is the orch of the worktree the pair was created from, so a prime batten over a pair created elsewhere would silence a driver whose relay goes to another orch.
- D4: a report path on a done notice, found as the newest file in the drive-reports directory. Worktree-Teardown removes the worktree holding it right after `Done`, so the path is unreadable by the time the orch sees it.
- D2: recording the cycle's start in `State`. The cycle's entry clears any pending request, so a request found later is already known to be newer, and the field would guard nothing.
- D3: a failed stat of the status file read as unchanged. A missing file would wait silently forever, where the per-bounce read found it absent and respawned.
- D7: leaving the batten window's `remain-on-exit` to a separate option call after creation. A batten that exits at once could die before the option is set and leave a dead window.
- D5: judging quiet from the driver's transcript. The driver idles inside a blocking step; the producer's agent is the one working.
- D5: counting a live Go-side wait as quiet. A gate is working; a hung gate is `verify-watchdog`'s concern.
- D6: removing the driver's parent messages for every run. A run no batten watches would then tell the orch nothing.
- D6: the agent deciding whether batten watches. The fact is Go's to render.
- D6: recording the watched fact as a seed param. The seed is write-once and compared for equality against what the child's own bootstrap writes, which refuses a seed with a param it does not write.
- D7: a separate `lyx batten start` verb. That is `orch-owns-the-loop`'s design; a flag on the existing verb is enough for the window.
- D7: keeping the window open after batten exits (`remain-on-exit`). Dead windows would pile up and blur the "runs in flight" list; batten's status and log keep the record.
- D7: leaving reed's enumeration on the session's current window. A second window, selected by hand, would be read as the strands' panes and reaped as untracked.
- Out of scope since round 1: the step watchdog, the Windows verify tree kill, Add's push retry and `rollbackAdd`'s branch deletion.

## Review rounds

Entries below name Decisions by the numbering of their round.
Burler round 1 withdrew the old D2 (turn ends read during compacting), so the old D3–D8 are D2–D7 from then on.

### Parent review — round 1

- F1 (BLOCKING, notice channel had no design): fixed. Added D4 (Run-Shed waits on the child, history only on state changes), D5 (event-timed notices carrying `since`, `history` and the report path), D6 (quiet from agent activity, an `api-error` notice) and D7 (a batten-watched driver messages only a question or a `parent_notice`), with scope, constraints and acceptance criteria. The old D5 (wait appended to the status-file quiet notice) is dropped; D6 excludes a live wait from quiet instead.
- F2 (BLOCKING, window per batten had no design): fixed. Added D8 (`lyx batten run <slug> --window` through a new reed op, orch role stencil updated), with scope, constraints and acceptance criteria; `orch-owns-the-loop` and `runs-overview` are listed out of scope.
- F3 (BLOCKING, watchdog and pair-create parts out of scope): fixed. Dropped the wait-overrun notice, the Windows job-object kill, Add's push retry and the rollback branch deletion, with their scope, constraints, acceptance criteria, risks and doc entries; D1–D3 kept.

## Question ledger

- Q1 (resolved, auto-pick): what hid the boundary from the compacting phase? Answer from the 2026-10-07 transcript: a queued operator message answered after the boundary; `ContextTokens` reads only the newest entry. Rationale: evidence, not assumption.
- Q2 (resolved, auto-pick): how should the compacting phase detect the boundary? `CompactedSince(ReadingTurnEnd, PhaseEnteredAt)`. Rationale: the board's second fix direction, robust to entries after the boundary.
- Q3 (resolved, auto-pick): what happens to a cycle request queued during a cycle? Same mode and made before its action took effect (the cycle's entry clears every earlier one): satisfied and removed when the action lands; anything else: kept. Rationale: avoids a redundant second cycle without dropping a different request or one made after the action.
- Q4 (resolved, auto-pick): how does batten stop ticking? The Run-Shed call waits inside with a 2 s check and returns only on a child state change, an arm event or a pause request; all Stucks exempt; the costly probes run every 30 s. Rationale: history becomes state changes; notices react within seconds.
- Q5 (resolved, auto-pick): how does a notice carry the report path when the report comes after the stop? Wait for the park marker or the driver's exit, at most 3 minutes, counting only a report written at or after the change. Rationale: the report is the notice's payload, and an older report belongs to an earlier stop.
- Q6 (resolved, auto-pick): what does quiet measure? Producer agents' transcript and event activity across the child's live shuttle runs, with live wait markers excluded; an API-error turn end notified after 2 minutes idle. Rationale: the 2026-10-07 API-error stall.
- Q7 (resolved, auto-pick): when does the driver still message the orch? A `parent_notice` relay or a question it cannot settle, only for batten-watched runs. Rationale: one channel per event.
- Q8 (resolved, auto-pick): how does a batten get its own window? `lyx batten run <slug> --window` through a reed op, in the reed session of the prime. Rationale: smallest surface; leaves `orch-owns-the-loop` its own verb.
- Q9 (resolved): scope after the operator's re-scope, reported by the parent review: orch and batten only; watchdog to `verify-watchdog`, pair create to `fabric-pushes`.
- Q10 (resolved, auto-pick, burler round 1): where does the batten-watched fact live? A marker in the run's scratch directory holding batten's pid, counted only while the pid is live. Rationale: the seed cannot carry it.
- Q11 (resolved, auto-pick, burler round 1): how does a second window coexist with reed's enumeration? The window is created detached and reed addresses its strand window by id. Rationale: reed lists the session's current window only, and treats every pane there as a strand or Selvage.
- Q12 (resolved, auto-pick, burler rounds 1 and 2): does an awaiting child get a batten notice? When its status carries no `parent_notice`; with one, the driver's relay is the one channel, and batten steps in only when no relay can have happened (Q18). Rationale: one channel per event, and the PR-Gate's awaiting carries no notice.
- Q13 (resolved, auto-pick, burler round 2): where does a pending state-changed notice live when the wait returns for the state change before its report exists? Derived from the child's status and the episode marker, sent by the wait of the next call; `Done` makes one last attempt. Rationale: the 3-minute report wait cannot hold the call open.
- Q14 (resolved, auto-pick, burler round 2): what does a done notice carry for the report? Nothing. Rationale: the worktree holding it is removed right after `Done`; the driver keeps its message for a `done` stop whose friction failed, which batten cannot know.
- Q15 (resolved, auto-pick, burler round 2): does the watched marker prove a notice destination? Batten writes it only while the orch state records a strand. Rationale: otherwise the silenced driver and the unqueued notice leave a stop announced nowhere.
- Q16 (resolved, auto-pick, burler round 2): where does reed's strand window id come from? The window holding the recorded Selvage pane, with today's current-window target as the fallback. Rationale: reed state records no window id, and one seam must serve every window-targeted op.
- Q17 (resolved, auto-pick, burler round 2): what does a failed stat of the child's status file do? It ends the wait and the next status read decides. Rationale: the wait must not turn a missing file into an unbounded silent wait.
- Q18 (resolved, auto-pick, burler round 3): does an awaiting child with a `parent_notice` ever get a batten notice? Yes, when its driver strand has ended or 3 minutes passed with no stop report, since no relay can have happened; a failed relay from a live, reported driver is bounded by the 10-minute fallback of Q25. Rationale: the Goal's "the orch hears each run stop".
- Q19 (resolved, auto-pick, burler round 3): what time is `since`? The status file's modification time at the check that first saw the state, kept in the episode key. Rationale: the child's history has no entry for a pause or a failure, and the file time is already what D3 stats.
- Q20 (resolved, auto-pick, burler round 3): when is the wait's baseline modification time taken? Before the call's status read, and any difference triggers the decode. Rationale: a write between the read and the first stat would otherwise leave the row on a stale state with no later read.
- Q21 (resolved, auto-pick, burler round 3): what targets the chained `remain-on-exit off`? The new window's exact name, with the id returned for later ops. Rationale: verified on tmux 3.6, an untargeted chained option lands on the current window.
- Q22 (resolved, auto-pick, burler round 3): is the driver's parent the orch the notices go to? Only for a batten run from the prime (A7), which alone writes the watched marker. Rationale: `QueueNotice` is wired to the prime's orch; a driver's parent is the orch of the worktree its pair was created from.
- Q23 (resolved, auto-pick, burler round 3): how often do the costly reads run? Driver liveness, agent activity, wait markers and the not-parked resume retry at most once per `notice_probe_s`; the file stats and reads on every 2 s check. Rationale: driver liveness is a reed status call.
- Q24 (resolved, auto-pick, burler round 3): what does the history say for a pause, a failed stat and a second pause? A pause carries `pause requested at <time>` so a repeat is its own entry; the failed-stat return is the other non-state entry, named in the Goal. Rationale: equal consecutive exempt Stucks fold.
- Q25 (resolved, auto-pick, burler round 4): what bounds an awaiting stop with a `parent_notice` whose relay failed? Batten suppresses its notice only while its watched marker stands, and sends it when the child is still awaiting with no decision 10 minutes after the report. Rationale: the report is announced by no notice, so a failed send recorded there bounded nothing; a duplicate is cheaper than a loss.
- Q26 (resolved, auto-pick, burler round 4): what guards the batten window's `remain-on-exit off` when a name is shared? The op reads the option back from the new window by id and kills a window that does not read `off`. Rationale: verified on tmux 3.6, the chained option on a duplicate exact name succeeds on the older window.
- Q27 (resolved, auto-pick, burler round 4): which panes name reed's strand window? A present Selvage pane, alive or a corpse, then a present strand pane, then the current window; the pane-generation read stays session-scoped. Rationale: reed keeps dead panes enumerable, and the generation check validates the ids the seam trusts.
- Q28 (resolved, auto-pick, burler rounds 4 and 5): where does `since` live? In batten's in-process episode state, across calls, pause returns and error returns until the state changes, read from the status file only when batten starts; the state-changed and driver-dead keys are the state and `since`, the quiet and api-error keys stay D5's activity. Rationale: the marker records only sent keys, so a pending notice has no stored first-sight time.
- Q29 (resolved, auto-pick, burler round 4): can the status decode miss a write? A write that leaves the modification time unchanged is caught by a decode at every `notice_probe_s` check, and the baseline precedes the read that defines the call's starting state. Rationale: the row has no time limit left to catch a stale state.
- Q30 (resolved, auto-pick, burler round 4): how long do the two retries run? The not-parked resume retry logs once per episode and ends when the driver parks or the state changes; the no-strand notice retry ends with its episode or at the `Done` return's last attempt. Rationale: each needs a stated end and a bounded log.
- Q31 (resolved, auto-pick, burler round 5): is the driver's parent the prime's orch whenever batten runs in the prime? No: the parent is the orch of the worktree the pair was created from, so the watched marker also requires the run's parent to be the prime's orch. Rationale: a prime batten can adopt or revive a pair created from another worktree, whose relay would go to that worktree's orch.
- Q32 (resolved, auto-pick, burler round 5): may the strand-window seam trust recorded pane ids after the generation check? Not alone: the check fails open on a probe failure and pane ids are server-wide, so a recorded id counts as present only in the session's own pane listing. Rationale: a stale id can name another session's pane.
- Q33 (resolved, auto-pick, burler round 5): what clock starts the quiet fallback? The later of the newest history entry and `since`. Rationale: a pause or a producer failure appends no history entry, so a resumed child would otherwise be quiet at once.
- Q34 (resolved, auto-pick, burler round 5): what do the window verb's other refusals name? `lyx batten run <slug>` in a terminal for a missing `new-window` or a failed call, and `lyx batten status <slug>` for the read-back refusal, since the other call's window may hold the live run. Rationale: every refusal of the verb names a way forward, and a terminal run would meet the run lock in the concurrent-call case.
- Environment: no tool failed during this interview.
