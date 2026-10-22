# Card 8 — Every writer and remover of the cycle request holds the state lock

**Edit:**
- `internal/orchengine#RequestCycle`
- `internal/orchengine#Watcher.tickIdle`
- `internal/orchengine#Watcher.removeSatisfiedRequest`
- `internal/orchengine#TestCycleRequest_WriteProbeClear`
- `internal/orchengine#fakeCreel`
- `internal/orchengine#fakeCreel.Hold`
- `internal/orchcli#orchCLI.requestCycle`
- `internal/orchengine#TestWatcher_ConfirmedClearRemovesOnlyASatisfiedClearRequest`
- `internal/orchengine/doc.go`

**Delete:**
- `internal/orchengine#ClearCycleRequest`

**Create:**
- `internal/orchengine#WithdrawCycleRequest`
- `internal/orchengine#RemoveCycleRequestIf`
- `internal/orchengine#takeMarkerUnlocked`
- `internal/orchengine#errMarkerChanged`
- `internal/orchengine#saveCycleStart`
- `internal/orchengine#TestCycleRequest_MarkerChangedDuringStart`

**Uses:**
- `internal/orchengine#CycleRequested`
- `internal/orchengine#CycleRequest`
- `internal/orchengine#saveStateForStrand`
- `internal/orchengine#Watcher.enter`
- `internal/orchengine#Watcher.renewHold`
- `internal/orchengine#Watcher.releaseHold`
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#watchEnv.injectHandoff`
- `internal/orchengine#TestCompact_CompletionRemovesOnlyASatisfiedCompactRequest`

**Intent:**
D5's second bullet: every watcher removal of `cycle-request` is one compare-and-remove of the mode and request time the watcher read, serialized with `lyx orch withdraw` (card 12) on the orch state lock (`Paths.StateLockPath`), so a withdraw answering `withdrawn: true` started no cycle, and a request written after a removal's read is never removed by it.

Marker primitives in `internal/orchengine/state.go`, each holding the state lock through `lock.AcquireWriteLock(p.StateLockPath)`, the lock `internal/state` takes for the state file:
- `RequestCycle` writes the marker under the lock with `fsx.AtomicWriteBytes`, a temp-file rename, so a locked reader never sees a truncated marker.
- `WithdrawCycleRequest` removes the marker under the lock and answers whether one was pending; an absent marker answers false and no error.
- `RemoveCycleRequestIf` removes the marker under the lock only when `takeMarkerUnlocked` finds it still holding `req`, and answers whether it removed it.
- `takeMarkerUnlocked` is the lock-free body the locked callers share: it reads the marker as `CycleRequested` does and, when its pending flag equals `pending` and its request equals `req` (same mode, `RequestedAt.Equal`), removes it if pending and answers true; otherwise it changes nothing and answers false.
  An unparseable marker reads as pending with the zero request on both reads, so a stale removal of one still matches.
- `ClearCycleRequest`, the unconditional unlocked removal, is deleted; its callers move onto the functions above on this card.

The cycle start: `saveCycleStart` persists the cycle's first phase and consumes the marker in one locked save, through `state.UpdateJSON` on the state file, inside whose callback it checks the strand exactly as `saveStateForStrand` does (answering `errStrandReplaced`, keeping the persisted `WatcherStopping`) and then calls `takeMarkerUnlocked(p, req, pending)` with what the tick read.
When that answers false and the trigger is `TriggerRequested`, it writes nothing and answers `errMarkerChanged`; for a hard or soft trigger it writes the phase and leaves the newer marker pending for the next idle.
`Watcher.tickIdle` keeps its order (marker read, trigger choice, render, `renewHold`) and replaces `enter` plus the unconditional clear with the same field updates `enter` makes, saved through `saveCycleStart` with the request and pending flag it read at the top of the tick.
On `errMarkerChanged` it releases the creel hold it took (`releaseHold`), posts nothing and stays idle, logging `orch: requested cycle not started, the request changed` at Info; the next idle tick reads the marker afresh, so a refresh written after a withdraw in that race runs as its own cycle.
The stale-request removal in `tickIdle` calls `RemoveCycleRequestIf` with the request it read, and `removeSatisfiedRequest` calls `RemoveCycleRequestIf` with the request it read and checked, so neither removes a marker written after its read.
`orchCLI.requestCycle` remembers the `CycleRequest` it wrote and, with no watcher live, removes it with `RemoveCycleRequestIf`, so it never removes a request another caller wrote meanwhile.

`fakeCreel` gains `onHold func()`, which `fakeCreel.Hold` runs before it records the call, so a test can change the marker between the tick's read and its locked save.

Tests:
- `TestCycleRequest_WriteProbeClear` drops `ClearCycleRequest` for `WithdrawCycleRequest`, answering true with a marker and false with none, and gains `RemoveCycleRequestIf` rows: a matching request is removed (true), a request with another `RequestedAt` or mode is left (false), and an absent marker answers false.
- `TestCycleRequest_MarkerChangedDuringStart` is new, a table whose rows set `onHold`:
  a requested cycle whose marker `WithdrawCycleRequest` removes during the hold stays idle with no note posted, the hold released, and no marker;
  the same with a new refresh `RequestCycle` writes after the withdraw stays idle on that tick, the new marker pending, and the next tick starts a requested cycle for it;
  a hard trigger (usage over the threshold, no marker at the read) with a refresh written during the hold enters handoff-requested and leaves that marker pending;
  a hard trigger with a marker pending at the read consumes it;
  a requested cycle whose `onHold` rebinds the state to another strand saves nothing, leaves the marker pending and keeps the persisted `WatcherStopping` the rebinding wrote.
  The ensuing removal of a request written during a hard start, by `removeSatisfiedRequest` when its mode is the cycle's and kept otherwise, is the case `TestWatcher_ConfirmedClearRemovesOnlyASatisfiedClearRequest` and `TestCompact_CompletionRemovesOnlyASatisfiedCompactRequest` already pin, since a request written during the start predates the cycle's effect.
  `TestWatcher_ConfirmedClearRemovesOnlyASatisfiedClearRequest`'s kept-request row, a request of the other mode, now ticks on through the reload to idle and asserts the kept request then starts its own requested cycle, so "otherwise it stays pending and runs at the next idle" is pinned whole.
- `internal/orchengine/doc.go` states the marker rule: every writer and remover holds the state lock, the cycle start compares and removes inside the save of its first phase, a changed marker leaves a requested start idle with its hold released and a hard or soft start entered with the newer marker pending, and the stale and satisfied removals compare and remove only the request they read.
- The existing `TestRefreshAndDistill_RequestTheirOwnMode` rows keep passing unchanged: with no watcher the marker is gone, with one it is pending.

**ImpactSummary:** The marker's four writers and removers (refresh/distill's request, the cycle start, the stale removal, the satisfied removal) move under the state lock with compare-and-remove; every production caller of `ClearCycleRequest` is on this card, and the test fixtures that call `RequestCycle` keep its signature.

**Commit:** 8: the cycle request is written and removed under the orch state lock
