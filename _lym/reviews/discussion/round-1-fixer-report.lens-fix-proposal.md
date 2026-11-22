```yaml
lens: fix-proposal
outcome: reported
```

Task: test the reviewer's proposals the fixer carried out as given, and look for cases where a proposal is wrong or breaks something elsewhere in the record.

Proposals examined: F1, F3, F4, F8, F9, F10, F11, F12, F13, F14, F15. The fixer marked F2 and F5 as fixed differently, so they are outside my lens.

**Findings**

1. **MEDIUM** — D5, first bullet: the `PairPush` mode "returns the owed `*PushFailedError` without writing its record … at the owed-class failures `pushClassified` records".
   - Summary: with no deferred record on disk, `recordsSideNotLanded` can no longer tell a deferred push. It misreads one as "unpushed past push" and then writes a Deferred record through `recordOwedRecordsPush`, which the same mode suppresses.
   - Evidence: `recordsSideNotLanded` in `internal/fabricengine/pushanchored.go` finds a deferral only through `readFailedPush(hubPath, failedPushSideRecords, branch)` with `record.Class == PushFailureDeferred && !record.LastSeen.Before(started)`, which is the record the push wrote. Under the F1 mode that record never exists, so the deferral is detected only by an in-memory error the record does not name.

2. **MEDIUM** — D5, dobby-tick bullet (F12): "a branch is ahead when … it has no tracking ref and holds at least one commit".
   - Summary: a records branch whose entry origin ended loses its tracking ref on the next fetch. It then reads as ahead forever, so the dobby starts a pusher on every tick for a branch the push deliberately skips.
   - Evidence:
     - Commit `f660ac101`, "Prune deleted remote branches on fetch so a cancelled slug can run again", drops the tracking ref of a branch deleted on origin.
     - `recordsSideNotLanded` returns the plain error "the entry of branch %s is ended on origin, so the branch was not pushed and is left to its teardown". That error has no failed-push class.
     - So no refused-push record ever stands to stop the restart, and the record states no other stop. A branch that was pushed once and then ended on origin is told apart from one never pushed by nothing in the new wording.

3. **MEDIUM** — D5, Worktree-Teardown bullet (F9): the push runs "before it takes the hub's prime lock".
   - Summary: the push now runs before `deps.Shutdown` ends the pair's loom session. A records commit the session makes before it shuts down is left unpushed and then removed with the pair.
   - Evidence:
     - `worktreeTeardownProducer.Call` in `internal/battenshed/teardown.go` runs `acquirePrimeLock`, then `p.deps.Shutdown`, then `p.deps.Remove`.
     - Under the new D1, the driver-stop `commit-records` verb commits and starts a pusher instead of pushing. A session stopping between the teardown's push and Remove commits records nothing pushes before the pair is gone.
     - This breaks D5's "the one step that removes local commits pushes them first" and the criterion "removes the pair only after its records and code reach origin".

4. **LOW** — D3, last bullet (F4): the pusher is registered under `runproc.HubOwner` with role `runproc.RolePush` so it outlives the run's process ends.
   - Summary: a pair's pusher now survives the run's teardown and `cancel --remove` by design, but the record never says how it exits once its worktree is removed underneath it.
   - Evidence: D3's exit protocol re-reads the local records tip, which errors on a removed worktree. `runproc.EndSubject` spares `RolePush` (`subjectPassSpared` in `internal/runproc/end.go`), so no end reaches it. No D3 or D5 sentence covers a gone worktree.

5. **LOW** — D5, first bullet and D8 (F1): no owed-class record is written for the pusher's retries.
   - Summary: an origin that stays unreachable leaves no failed-push record, so nothing outside the logs shows the operator that a pusher has been backing off for hours.
   - Evidence: `FailedPushFields` in `internal/fabricengine/failedpush.go` renders the hub's failed-push records as the verb's rows. With the mode on, the pusher's owed state never appears there. Before, a transport record also carried a notice through `recordPushFailure`'s notifier.

No other proposal yielded a case: F3, F8, F10, F11, F13, F14 and F15.

Tool calls: none. My context already held the code spans and the recent commit list these findings cite.
