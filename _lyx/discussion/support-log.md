# Support log — lifecycle-waits

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry as it stood at the start (orch compact reload, step watchdog, issue #433) and the touching entries: `run-recovery` (done; built the wait markers and the verify timeout), `run-pushes` (done), `bugfix-fabric` (done), `parent-review-hardening` and `orch-owns-the-loop` (notes).
2. Compaction: read `tickCompacting` and `tick()` in `internal/orchengine/watcher.go` and the Claude transcript readers in `internal/shuttleengine/claudeengine/usage.go`.
   The board's diagnosis says the turn ends after compaction are not read; the transcript readers reopen the file at read time, so the stale turn end alone would not hide the boundary.
   Checked the hub orch transcript of 2026-10-07: the `compact_boundary` at 10:06:41Z is in the same file and is followed at 10:06:42Z by a queued operator message and its answer.
   `ContextTokens` returns only the newest qualifying entry, so that assistant entry hid the boundary.
   Fix: use `CompactedSince` with `PhaseEnteredAt` (D1), and keep the reading current through compacting (D2).
3. The stale `distill` side effect: `tickIdle` removes a request older than the handoff timeout; a request made during a ten-minute stalled cycle aged out.
   With D1 the cycle finishes in about a minute, so the request would instead trigger a second compaction; picked D3.
4. First draft also designed the step watchdog, the Windows verify tree kill, and #433's push retry and rollback.
   The parent review (round 1) reported that the operator re-scoped the entry during Discussion to orch and batten only: those parts moved to the note `verify-watchdog` and the task `fabric-pushes`, and two sections were added: one notice channel to the orch, and a window per batten. Re-read the entry with `--body`.
5. Notice channel: read `internal/battenshed/innerrun.go`, `notice.go`, `batten-recipe.yaml`, the notice queue in `internal/orchengine/notice.go` and its watcher delivery, and the driver stencil's `## Notifying the parent`.
   Run-Shed bounces every 30 s by design (self-route with 1440 counted bounces); every bounce writes a history entry, folded only when identical and exempt.
   Picked a wait inside the call that returns only on a child state change, an arm event or a pause request, with a 2 s in-call check (D4).
6. Notice content: the stop report exists only after the driver writes it and parks (the park marker holds its path), so the state-changed notice waits for it up to 3 minutes; every notice gains `since` and `history` (D5).
7. Quiet: the status file moves only at step ends. Checked a 2026-10 transcript for an API-error turn: a synthetic main-chain assistant entry with `isApiErrorMessage: true`, `error: server_error` and the error text.
   Picked an activity reading in shuttle (transcript and events of live runs), a quiet condition on agent idleness with live wait markers excluded, and an `api-error` condition after 2 minutes idle (D6).
8. Driver messages: the driver messages the parent at every escalation and on friction failure; with batten owning state changes, a batten-watched driver messages only a `parent_notice` relay or a question (D7).
9. Window per batten: the orch role stencil starts battens with `setsid nohup`; reed owns tmux and the pane-binary chokepoint. Picked `lyx batten run <slug> --window` through a new reed op (D8).

## Rejected alternatives

- Compaction: re-reading `ContextTokens` through each turn end read during compacting, without `CompactedSince`. Any assistant entry after the boundary still hides it, which is the observed failure.
- Compaction: a transcript poll for any boundary newer than the phase entry, separate from `CompactedSince`. `CompactedSince` already does this with the transcript path from the held turn end.
- D3: leaving a same-mode request in place. It would start a second compaction a minute after the first.
- D3: dropping every request made during a cycle. A clear requested during a compact cycle asks for something the cycle did not do.
- D4: keeping the bounce loop and only shortening `poll_interval_s`. History would grow faster, the opposite of what the operator asked.
- D4: a file-notification library (fsnotify) for the child's status file. A new dependency with platform quirks for a gain of under 2 s.
- D4: keeping a wall-clock limit on the running watch. A long run has no natural limit, and a limit hit ends nothing useful; the quiet notice reaches the orch instead.
- D5: sending the state-changed notice at once without the report path. The board asks that the notice carry the report, which is written after the stop.
- D5: superseding older queued notices for the same slug in the orch's queue. `since` and `history` already make a stale notice recognisable, and changing the queue's contract is not asked for.
- D6: judging quiet from the driver's transcript. The driver idles inside a blocking step; the producer's agent is the one working.
- D6: counting a live Go-side wait as quiet. A gate is working; a hung gate is `verify-watchdog`'s concern.
- D7: removing the driver's parent messages for every run. A run no batten watches would then tell the orch nothing.
- D7: the agent deciding whether batten watches. The fact is in the seed; Go renders it.
- D8: a separate `lyx batten start` verb. That is `orch-owns-the-loop`'s design; a flag on the existing verb is enough for the window.
- D8: keeping the window open after batten exits (`remain-on-exit`). Dead windows would pile up and blur the "runs in flight" list; batten's status and log keep the record.
- Out of scope since round 1: the step watchdog, the Windows verify tree kill, Add's push retry and `rollbackAdd`'s branch deletion.

## Review rounds

### Parent review — round 1

- F1 (BLOCKING, notice channel had no design): fixed. Added D4 (Run-Shed waits on the child, history only on state changes), D5 (event-timed notices carrying `since`, `history` and the report path), D6 (quiet from agent activity, an `api-error` notice) and D7 (a batten-watched driver messages only a question or a `parent_notice`), with scope, constraints and acceptance criteria. The old D5 (wait appended to the status-file quiet notice) is dropped; D6 excludes a live wait from quiet instead.
- F2 (BLOCKING, window per batten had no design): fixed. Added D8 (`lyx batten run <slug> --window` through a new reed op, orch role stencil updated), with scope, constraints and acceptance criteria; `orch-owns-the-loop` and `runs-overview` are listed out of scope.
- F3 (BLOCKING, watchdog and pair-create parts out of scope): fixed. Dropped the wait-overrun notice, the Windows job-object kill, Add's push retry and the rollback branch deletion, with their scope, constraints, acceptance criteria, risks and doc entries; D1–D3 kept.

## Question ledger

- Q1 (resolved, auto-pick): what hid the boundary from the compacting phase? Answer from the 2026-10-07 transcript: a queued operator message answered after the boundary; `ContextTokens` reads only the newest entry. Rationale: evidence, not assumption.
- Q2 (resolved, auto-pick): how should the compacting phase detect the boundary? `CompactedSince(ReadingTurnEnd, PhaseEnteredAt)`, with turn ends read during compacting stored as the reading. Rationale: the board's second fix direction, robust to entries after the boundary.
- Q3 (resolved, auto-pick): what happens to a cycle request queued during a cycle? Same mode: satisfied and removed at completion; other mode: kept. Rationale: avoids a redundant second cycle without dropping a different request.
- Q4 (resolved, auto-pick): how does batten stop ticking? The Run-Shed call waits inside with a 2 s check and returns only on a child state change, an arm event or a pause request; all Stucks exempt. Rationale: history becomes state changes; notices react within seconds.
- Q5 (resolved, auto-pick): how does a notice carry the report path when the report comes after the stop? Wait for the park marker or the driver's exit, at most 3 minutes. Rationale: the report is the notice's payload.
- Q6 (resolved, auto-pick): what does quiet measure? Producer agents' transcript and event activity across the child's live shuttle runs, with live wait markers excluded; an API-error turn end notified after 2 minutes idle. Rationale: the 2026-10-07 API-error stall.
- Q7 (resolved, auto-pick): when does the driver still message the orch? A `parent_notice` relay or a question it cannot settle, only for batten-watched runs. Rationale: one channel per event.
- Q8 (resolved, auto-pick): how does a batten get its own window? `lyx batten run <slug> --window` through a reed op, in the reed session of the prime. Rationale: smallest surface; leaves `orch-owns-the-loop` its own verb.
- Q9 (resolved): scope after the operator's re-scope, reported by the parent review: orch and batten only; watchdog to `verify-watchdog`, pair create to `fabric-pushes`.
- Environment: no tool failed during this interview.
