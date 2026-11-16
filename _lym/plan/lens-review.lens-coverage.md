```yaml
lens: coverage
outcome: reported
```

Coverage lens: does each decision, acceptance criterion and new branch have a card and a test row that would fail if it broke?

I found 17 gaps, all MEDIUM or LOW. No BLOCKING findings.

**MEDIUM**

1. **D1, pause between a transition's commit and its push (card 8).** No row writes a pause after the commit and asserts both halves of the criterion: the first push attempt still lands, and the next producer does not start. `TestNew_OwedPushRetriesBehindTheGate` only asserts that the admission is released before the first retry. The criterion is "a pause written between a transition's commit and its push leaves that transition on origin, and the run's next producer does not start".

2. **D1, owed push turns pausable (cards 3 and 8).** No row joins the two halves of "turns owed, and a pause written then holds the process's network calls". Card 3's row stops at the `teardown_lock` record. Card 8's tier0 row checks a fake `Admit` release and never checks that a pause then holds the retries.

3. **Card 8, `fabricengine.IsPushOwed`.** No row covers its branches: a `*PushFailedError` of class transport, one of class teardown_lock, `ErrPushLockBusy`, `ErrPairPushRunning`, and a `not_fast_forward` failure reading not owed. The seam retries or halts on this one function, so a misclassification ships the wrong behaviour unseen.

4. **Card 3, teardown wait that succeeds.** D1 says "A transition's push that meets a held teardown lock waits for its release". The only planned row lets the wait run out. Nothing releases the lock within `TeardownWait` and asserts that the push then lands.

5. **D1 criterion 7, the anchor change (card 5).** Card 5 argues the criterion through the `CoalescePushRequest` row ("which D1's anchor-change criterion rests on"). No row drives `internal/fabricengine#AnchorChange.Finish`'s push against a pair that was taken over. `TestAnchorChange` is not extended.

6. **D1 criterion 8, the fixer attempt counts (card 8).** Card 8 says the counts reach origin because the run's status pathspec carries them, and no row checks it. The counts are written inside `gateAndFix` during the Finalize producer. Nothing shows they are committed and pushed before the next producer, which is what the criterion requires.

7. **D1 criterion 9, an owed pause intent landing (card 9).** The criterion says the pause intent is on origin "once origin answers". Card 9 logs the owed push and returns nil, and nothing in the plan retries it. The planned row only lands the intent "on the next pause once origin is restored", which is a different trigger. The record's arbiter entry leaves an owed push to D4's catch-up, which is another part. The plan neither says so nor adds a row.

8. **D1, `lym batten cancel` offline (cards 8 and 9).** Card 8 moves `commitCancelRecord` onto a synchronous `Push`, but says nothing of an owed push there. `endRunPhase` calls `CommitRecord` before `EndForCancel`, so an owed error would stop the cancel before it acts. The record says cancel acts here at once and owes its push. Card 9 covers only `clearRunPhase`'s `PushRecords`, and no row drives the intent commit's push while it is owed.

9. **D2 criterion 4, the old owner's local parent branch (cards 6 and 11).** `pairteardown.HandOver(prime, slug, entry, push RunPush)` takes no parent pair, and nothing says who fills `HandOverRequest.Parent`. A Finalize hand-over therefore never reaches `archiveParentMerge`. Card 11's `TestFinalize_OwnerChecks` row asserts only "hands over and pushes nothing", never "level with its upstream, merge under an archive tag".

10. **D2 criterion 4, the adopter's side (card 11).** No row covers the adopter merging again and origin holding the work once.

11. **D2, the dropped-intent notice (card 6).** "posts one notice naming the machine and any pause or cancel intent it dropped" has no source: `HandOver` and `PushRun` take no `DroppedIntents`, and nothing computes them. No row asserts the notice or the intent it names.

12. **D2 criterion 9, a Finalize cut between merge and parent push (card 11).** Card 11 argues "the re-step's merge reads up to date and pushes again before any transition", but no row covers it. `TestFinalize_TransportPushFailure_ReturnsClassifiedError` is not among card 11's edits.

13. **Card 7, `resetUnnamedTip`.** The planned guard row covers only a tip that no transition names. No row covers "a tip a transition names keeps the rejection as today", nor the "descends from the local tip's merge base" condition. That condition is also looser than the record's operand, "a tip that no transition of the run on origin names".

**LOW**

14. **Card 12, branches with no row:**
    - `ReturnToPreviousTransition` returning false when there is a single transition.
    - A give-up push that fails or is owed, where the next tick retries.
    - A `settleCodeTip` error, which leaves the run out of the tick.

15. **Cards 5, 6, 9 and 13, branches with no row:**
    - `takeoverStop` with a zero `MachineNames` or an empty slug.
    - A hand-over mark naming a different entry (another machine or `At`), which should hand over again.
    - `ReadUpstreamTakeover` with no upstream.
    - Failures inside `HandOver` (process end, `HandOverTips`, removal).
    - Tender's non-owed push error being returned.
    - Card 13's unreadable verdict leaving `Owes` empty.

16. **Card 8's `TeardownWait` on the batten seam is inert.** Card 8 passes `TeardownWait: StatusPushLockWait` to batten's prime push. `statusPushRequest` sets no teardown slug for the prime (`internal/fabricengine/spawn.go:156-169`), so the wait never applies. Separately, `IsPushOwed` counts `ErrPairPushRunning` as owed, but a zero-wait caller writes no failed-push record. `ReadRun`'s `PushOwed` (card 7) then misses that owed state.

17. **D2, `EndDobby` (card 6).** "with the dobby end unset when the caller is that dobby's own run" adds `RunPush.EndDobby`, but no caller is named and no row covers it.

Further tool calls made: none; my context held everything this lens needed.
