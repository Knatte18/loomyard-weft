```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: findings.

1. **BLOCKING — D5 rebase refusal (card 11 step 3).** After the named rebase, nothing ever pushes the newer landing.
   - The refusal writes a `held_back` record whose way forward is only "rebase in the parent's worktree".
   - Card 11's `starter.start` spawns only a settle-only pusher while that record stands. Settle-only runs step 1 alone, so it never re-reads the owed tips or pushes them.
   - A rebase clears no record. No full pass ever runs again, and the D5 criterion "after which the next pass pushes and settles the newer entry" cannot happen in production.
   - `TestParentPush` hides this: it drives the next pass directly over fake `ParentDeps`, bypassing the start gating.
   - Fix: name a push after the rebase (`git push` in the prime's worktree, or `lym fabric push` for a beam parent) so the settle clears the record. Or clear or re-judge the record when a pass finds no landing commit standing unrecorded.

2. **BLOCKING — D5 "leave the run to its owner" (card 11 step 4, run owned elsewhere).** This way forward is a dead end.
   - Its promise is that "once the owner lands it, the next pass drops the landing".
   - The drop is step 3, but while the run's `held_back` record stands only settle-only pushers start (card 11, `starter.start`).
   - Settle-only does step 1, which settles a landing only when origin's parent holds its recorded `Tip`. An owner's re-landing puts the work there under another commit, so step 1 never fires.
   - The landing entry, the `held_back` record and the pair hold stand forever. The D3 all-clear never posts, and card 19's teardown refusal never lifts.
   - Fix: let the settle-only pass also run the step 3 drop read and clear the record.

3. **BLOCKING — D5 pair hold after a drop (cards 4, 9, 11, 19).** The plan releases the hold when D5 says it must outlive the entry.
   - D5 says: "the entry's pair hold outlives it, so no push sends the held records commit or any after it to a branch, and the hold goes with the pair".
   - The plan's Shared Decision makes the hold last exactly as long as the entry. `HeldRecordsCommit` reads only standing landings (card 4), and card 11's drop removes the entry.
   - After a drop on a run this machine owns, the pair's next `PushRun` sends the held records commit: the pusher, a catch-up, cancel, or the removal's pushes.
   - Card 19's sentence "the held records commit and any after it go to no branch" is false under card 9's mechanism.
   - Fix: the drop must leave a hold marker that lasts until the pair is removed.

4. **MEDIUM — the `held_back` refusal for a tip carrying an unrecorded landing commit (card 11 step 4, last sentence).** It is "refused the same way, naming the commit" but gives no next step.
   - It names neither the rebase that drops the commit nor the push after it.
   - Card 10's `PushFailureHeldBack` returns the record's own `WayForward`, so an empty one gives the operator no way out.
   - As in finding 1, nothing clears the record.

5. **MEDIUM — stale wait record (cards 2 and 3).** No recovery step exists for a record a dead driver leaves.
   - A driver killed mid-wait leaves its `DriveWait` record, and card 2 removes it only "when the wait returns".
   - When a new driver takes the driver lock, `ReadDriveWait` reads the stale record as `standing` under a held lock, so the run reads as Waiting.
   - From then on card 3's run watch owes no line for that run, so a later real stop, such as a hand-back, mails nothing.
   - The `waiting` key in `lym loom status` and D3's summary list a wait that is not happening.
   - Fix: the driver removes any wait record at start, before its first step, and a card 3 row pins it.

6. **LOW — card 13's refused-push halt.** It is a blocked halt "naming the refusal's way forward" but omits the resume that lifts it.
   - Once the way forward lands and the next tick's settle clears the record, the run stays halted until someone resumes it.
   - The reason should end with `lym loom resume` in the task worktree, as `ownerRerun` and `gotoResumeWayForward` do in `internal/landingshed/finalize_owner.go:23`.

7. **LOW — card 11's way forward for a beam parent owned elsewhere** (`lym adopt <beam>`, then `lym fabric push`).
   - This progresses: a landing `lym fabric push` clears the branch's record through `pushclassified.go:474`.
   - For the prime, though, `git push` clears nothing itself. The record clears only through the landing target's settle-only pusher, and only the dobby tick's `StartIfAhead` starts that pusher (card 6).
   - No card's Intent states that the tick restarts the landing target while its record stands. Card 11's `starter.start` text implies it, but card 6's tick restart through `StartIfAhead` reads only "a landing entry stands".
   - Fix: say in card 11 that `StartIfAhead` admits a refused target as settle-only, so the D5 "prime way forward clears the record" criterion holds through the real start path. `TestParentPush` covers only the start decision.

Calls made:
- `grep ClearPushFailure` callers: to learn whether a landing push clears a `held_back` record, which my context lacked.
- `grep sheddriver` in the battencli, battenshed and beamcli packages: to confirm batten runs under the Go driver, so card 16's `git-transport` Land stop is waited out. It is (`internal/battencli/drive.go`).
- `grep` for transient handling in `internal/dobbyengine`: no match.
