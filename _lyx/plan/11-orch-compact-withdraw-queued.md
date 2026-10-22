# Card 11 — A compaction seen while `/compact` is still queued withdraws it

**Edit:**
- `internal/orchengine#Watcher.tickCompacting`
- `internal/orchengine#TestCompact_CompletesOnBoundaryAfterEntryAndIdle`
- `internal/orchengine/doc.go`

**Create:**
- `plan:internal/orchengine#Watcher.withdrawQueuedCompact` -> `func (w *Watcher) withdrawQueuedCompact(st *State) error`
- `plan:internal/orchengine#TestCompact_BoundaryWithdrawsAStillQueuedCompact` -> `func TestCompact_BoundaryWithdrawsAStillQueuedCompact(t *testing.T)`

**Uses:**
- `internal/orchengine#Watcher.resetForOtherStrand`
- `internal/orchengine#ClearResumeNoticed`
- `internal/orchengine#fakeCreel.Withdraw`
- `internal/orchengine#fakeCreel.InjectOutcome`
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#watchEnv.reachCompacting`
- `internal/orchengine#watchEnv.landBoundary`
- `internal/orchengine#watchEnv.noticedResumedEntry`
- `internal/orchengine#watchEnv.setOutcome`

**Intent:**
D7's watcher part of issue #828: the creel journal of 2026-10-22 shows the `/compact` inject requeued after the session had compacted on its own; once the compacting phase finds a qualifying boundary, a `/compact` still queued would compact the session a second time.

`tickCompacting`, on a qualifying boundary and before it stores the reading and completes the phase, calls `withdrawQueuedCompact` when the `/compact` inject has no outcome, the record's operand, read from creel rather than from the state:
`st.Pending` is set, `st.PhaseInjected` is false, and either the pending step has no id yet (creel never answered its send) or `creel.InjectOutcome(st.Pending.ID)` answers queued or sending.
The boundary is checked before `stepOutcome` runs, so the state alone would read an inject creel typed in the same tick as still queued; reading the outcome keeps a typed `/compact` from a needless receiver-wide withdraw.
A failed outcome read goes through `creelFailed` like a failed withdraw.
`withdrawQueuedCompact` withdraws the watcher's queued creel items to the orch strand by name (`creel.Withdraw`), and clears the resume-noticed mark of a hub-pause resume line the withdraw removed, as `resetForOtherStrand` and `retryFreshLaunch` do: with a pause record told, it hands the withdrawn list, plus a `pauserecord.ResumeLine` entry when `st.ResumeLineWithdrawn` is set, to `ClearResumeNoticed`, and on a clear failure keeps `ResumeLineWithdrawn` set and saves it.
A failed withdraw goes through `creelFailed`, leaves the phase compacting with the boundary unconsumed, and the next tick, which searches the transcript for the boundary again, retries it.
On success it clears `st.Pending` and the phase completes as today: the reading, the cycle count, the baseline, `removeSatisfiedRequest` and the reload.
Bound: the withdraw is receiver-wide, as every watcher withdraw is, so it also removes any other queued watcher item to the orch, which the reload that follows re-queues as it needs.

`internal/orchengine/doc.go`'s compacting paragraph gains the withdraw.

`TestCompact_BoundaryWithdrawsAStillQueuedCompact` is new: with the fake creel not landing injects, `reachCompacting` leaves `/compact` sent with an id and queued (card 10 keeps the send in the gate's tick; `setOutcome` answering queued), a boundary lands, and the next tick withdraws (`withdraw:<orch>` in the fake's calls) and moves on to resuming; a row with the outcome scripted injected and a boundary in the same tick makes no withdraw; with a noticed resumed pause entry (`noticedResumedEntry`) and the fake's `Withdraw` answering a `pauserecord.ResumeLine` subject, the entry's `ResumeNoticed` is cleared; with the pause record made unwritable so `ClearResumeNoticed` fails, the tick keeps `ResumeLineWithdrawn` set in the saved state; with `Withdraw` failing once, the tick leaves the phase compacting and records the failure, and the following tick withdraws and completes.
`TestCompact_CompletesOnBoundaryAfterEntryAndIdle`, where the compact was typed before the boundary, keeps passing and gains the assertion that no withdraw was made.

**ImpactSummary:** One extra creel call on the compacting phase's completion when its inject is still queued; the phase's other exits and every other caller are unchanged.

**Commit:** 11: a compaction seen while /compact is queued withdraws the inject
