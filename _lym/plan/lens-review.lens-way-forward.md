```yaml
lens: way-forward
outcome: reported
```

Way-forward lens review of the landing-queue plan: per refusal, the next step it names and whether that step can succeed from the state the refusal leaves.

**Refusals that progress:**
- **Card 1:** a backstop seam error keeps the mark, and the next lock taker runs the backstop again.
- **Card 7:** a failed `joinQueue` write is an error, and the driver re-steps the landing.
- **Card 7:** a backstop error under the lock ends `Call`, and the re-step reruns the backstop.
- **Card 7:** `conflict` and `mergeNeedsSession` free the lock, merge with no lock, then take the lock at the front.
- **Cards 7 and 9:** a `failed` run, or a culprit leader whose fixer is spent, halts with "fix … then `lym loom resume`". On resume the run joins again, keeps `failed` and lands alone.
- **Card 9:** a batch gate that is unjudged (runner crash or memory-pressure stop) halts with `lym loom resume`, which leads the batch again.
- **Card 10:** a dirty detached worktree is Stuck with "clean them, then `lym loom resume`". After cleaning, the shed restore puts it back on its branch.
- **Card 10:** a detached worktree with no marker is Stuck with "check the task branch out, then `lym loom resume`". After that the run goes on.
- **Card 8:** a member left out is met by its own Finalize under the lock.

**Findings:**

1. **MEDIUM. Card 4 pusher hold-off (D5): the hold-off has no end except a resume, and the pusher keeps looping through it.**
   - Card 4 makes `pushClassified` return nil on the code side while a marker stands. `readLocalBranch` still reads the task branch, so a branch ahead of origin stays ahead.
   - `internal/runpusher/run.go:212-275` (`pushUntilLevel`) loops while `deps.Ahead()` is true, and `recordAttempt(nil)` records a success. The pusher therefore keeps "pushing" at the base interval forever and writes no failed-push record that would name the cause.
   - The marker outlives a dead leader. Card 4's `RestoreDetached` removes it, stale or not, only on the run's resume. A halted run that is never resumed holds its code push indefinitely.
   - Proposed fix: a held-off push returns an owed error of the deferred class, which the pusher waits out and does not spin on. The `Bound` line should name `lym loom resume` as the way that ends the hold.

2. **MEDIUM. Cards 4 and 10: a marker that names no existing branch leaves no step that succeeds.**
   - Card 4 says `parentHoldsTaskWorkAt` errors naming the marker. Finalize turns that error into the owner-check halt `ownerHoldsReadReason` (`internal/landingshed/finalize.go:675-677`), whose way forward is "re-run `lym loom resume`".
   - On resume, card 10's `Shed.Restore` runs `RestoreDetached`. Its `CheckoutBranch` of the missing branch fails, and that error is "returned from `Run` or `Step` as it stands". So every resume fails the same way.
   - No message says to remove the marker or check a branch out.
   - Proposed fix: name the marker path and the step that clears it in both the restore error and the read error.

3. **MEDIUM. Card 10 `Shed.Restore` error: it names no way forward.**
   - `refuseUnderRunLock` wraps its error in `ErrLockedCheckRefused`, which keeps the check's own way forward (`internal/shedengine/run.go:594-603`, `shed.go:70-76`). Card 10 instead returns restore errors bare.
   - The restore errors are a failed checkout, a failed leftovers removal and a marker read. Each blocks every row's resume and `lym loom goto`, and no step is named.
   - Proposed fix: wrap the error with the worktree, the marker path and "check the task branch out in the task worktree, remove the marker, then `lym loom resume`".

4. **MEDIUM. Card 10 Bound: a run resumed at a row other than Finalize, with a dirty or unmarked detached worktree, gets no stop.**
   - D5 puts both Stucks on "the run's resume, which every row's resume and `lym loom goto` pass through". Card 10 puts them in `Finalize.Call` alone, and the shed closure returns nil for both states.
   - The Bound claims "that row's own clean-tree check" covers it. Only the landing rows (Parent-Align and Finalize, through `verifyGate.clean`) are known to run such a check, so other rows, Webster for one, would run with a detached HEAD.
   - Proposed fix: return both Stucks from the shed restore itself, with D5's way forward.

5. **MEDIUM. Cards 3 and 8 guarded move: an error from `MoveCodeTo` loses today's way forward.**
   - Card 3's `MoveCodeTo` errors on tracked dirt, a detached parent head, or a tip that does not descend from the base. Card 8 does not say how the leader ends on that error.
   - Today a dirty or detached parent is Stuck with `DirtyParentRefusal` or `dirtyParentReason` and their goto/resume way forward (`internal/landingshed/finalize.go:810-844`).
   - As written the error ends `Call` as a plain error, which the driver re-steps with the parent still dirty, and the mark is written before the move. `syncParent` checks for dirt only before the gate, so dirt that appears during the gate reaches this error.
   - Proposed fix: map the dirt and detached-head cases to the existing Stuck reasons, and say the mark written before the failed move is removed.

6. **LOW. Card 8: a move that returns false leaves the mark standing.**
   - Card 8 writes the mark before `MoveCodeTo` and then rebuilds when the move returns false. It never removes or replaces that mark.
   - The rebuild may end in landing alone after the cap. Card 1 does not say whether `WriteMark` replaces a standing mark.
   - The stale mark is cleared only at the next lock taker's backstop. Card 8 should remove the mark on a false move, or card 1 should state that `WriteMark` replaces it.

7. **LOW. Card 8: `Detach.Restore` failing during a live hold has no stated ending.**
   - Card 8 runs `Restore` and then `RemoveMarker` "after its last restore", but says nothing for a restore that fails, for example `CheckoutBranch` refusing untracked dirt the gate left.
   - If the marker is removed anyway, the next resume reaches the no-marker Stuck and its manual checkout step, not the automatic restore.
   - Proposed fix: on a failed restore keep the marker and end Stuck with card 10's dirty-path way forward.

8. **LOW. Card 11 backstop error: it names only the queue directory.**
   - A persistent backstop failure, a `runpusher.RecordLanding` or ref-read error, makes every `lym fabric anchor` and `lym fabric pull --hub` over that branch fail on each re-run.
   - "Its re-run runs the backstop again" cannot succeed while the cause stands, and nothing names how to clear the mark.
   - Proposed fix: name the mark file and the step that clears it.

Calls made beyond the inherited context:
- `grep` of `internal/runpusher/run.go`, to find how the pusher's loop treats a push that leaves the target ahead, which the context lacked.
- `sed -n 184,300p internal/runpusher/run.go`, to read `pushUntilLevel` and confirm it keeps looping when a push returns nil while the target is still ahead.
