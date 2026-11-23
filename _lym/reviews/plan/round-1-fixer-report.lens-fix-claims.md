```yaml
lens: fix-claims
outcome: reported
```

Task: check that the fix's added or rewritten claims hold against the code at HEAD (fix-claims lens).

**MEDIUM: card 2, `TipsOnOrigin` paragraph ("A pair whose run the ended set … reads ended reads held on both sides, whatever its unpruned tracking refs hold")**
- **Summary:** the fix keys "held" on the run being ended alone. It drops the review's condition that origin no longer has the branch, so this states a guarantee the code may not give.
- **Evidence:** `readEndedSet` (`internal/fabricengine/replay.go:117`) marks a slug ended when the ledger shows the run done: `Ended: isDone(task)` (`internal/boardengine/replay.go:286`). It has no tie to whether the branch exists on origin. The batten commit seam writes the board status on every row (`internal/battencli/commitstatus.go:186`).
- **Risk:** if a run reads done on origin's board before or during Worktree-Teardown, `Level` reads held no matter what the tracking refs hold. A tip that moved after the push would then be removed without being pushed, the exact case card 10's re-read of `Level` exists to catch.
- **Not confirmed:** I did not settle whether the done status lands before Worktree-Teardown runs. The sentence still asserts more than the review asked for.
- **Same widening in `RunCommitsAhead`:** its added sentence makes "a pair's records branch whose run the ended set reads ended" read level. That also covers a branch origin still holds, whose unpushed commits the dobby then never restarts a pusher for.

**MEDIUM: card 2, `RunCommitsAhead` ("so a pusher whose push would end in `ErrEntryEnded` is never started for it")**
- **Summary:** "never" does not hold. The read uses the local tracking ref of origin's Bolt branch, which only a fetch updates.
- **Evidence:** `readEndedSet`'s doc says "The ref holds origin's Bolt tip as last fetched". Only the push path's `goneEntryEnded` (`pushclassified.go:759`) and `fetchBoltBranch` (`:918`) fetch it. Before such a fetch, a stale ref does not show the run ended, so the first tick still starts a pusher.
- **Correct claim:** later ticks read level once a push has fetched the Bolt branch.

**Claims checked and found to hold:**
- Card 8: the condition `!inputs.OwnedElsewhere && view.PairPresent` exists at `dobbyreads.go:259`.
- Card 2: the code side's gone-branch check fetches the Bolt branch (`awaitOrigin` → `goneEntryEnded`), and `checkOrigin` returns nil on `view.ended`.
- Card 3: the cancel and re-adopt record pushes and `pushOwedRunCommits` are prime pushes that name a slug (`commitstatus.go:171-173`, `handover.go:184-185`).
- Card 3: with `Pair` nil, no pair archive runs (`handover.go:88`).
- Card 1: the rows `Deferred_HeldWeftWriteLock`, `PrimeRecordsPushOwed_OriginLookupTransportFailure`, `PrimeRecordsPushOwed_BackoffSpent` and `TeardownLockHeldPastTheWaitOwesTheRecordsPush` all exist.

Tool calls:
1. Read `awaitOrigin`, `goneEntryEnded`, `recordsSideNotLanded` and `fetchBoltBranch`, to check card 2's Bolt-fetch claim; my context lacked their bodies.
2. Read `readEndedSet`'s semantics and grepped the teardown docs, to see what "ended" means; my context lacked them.
3. Grepped where `LedgerRun.Ended` is set, to tie "ended" to board status; my context lacked it.
4. Grepped `IsDoneStatus` and `SetRunStatus` callers, to judge when a run reads done relative to teardown; my context lacked them.
