```yaml
lens: coverage
outcome: reported
```

Coverage lens findings, from the plan files as my context holds them. I made no further tool calls.

1. BLOCKING — D5 and D13, dropped landing: the hold ends with the entry, but D5 says it must outlive it.
   - D5 says: "the entry's pair hold outlives it, so no push sends the held records commit or any after it to a branch, and the hold goes with the pair."
   - Card 4 and the shared decision "a landing entry is the hold" make the hold last exactly as long as the entry. Card 9's `PushRun` reads `pushledger.HeldRecordsCommit`, which reads only standing landings.
   - Card 11 step 3 removes the dropped entry, so the hold lifts. `HandOverOwnedElsewhere`'s pushes, and a later `lym fabric remove`, would then send the held commit.
   - Card 19 claims "the held records commit and any after it go to no branch" but names no mechanism.
   - Card 11's `TestParentPush` drop row runs over a fake `HandOver` seam, so no row would catch this. D5's criterion "no branch on origin receives the held records commit" goes untested.

2. BLOCKING — D5 hold, card 9: `PushRun` sets `RecordsHold` from the slug alone, whatever worktree `l` it pushes.
   - `PushRun(prime, prime, slug, …)` pushes the prime's records. It is called from `internal/battencli#pushOwedForCatchUp`, `internal/battencli#hubCancelDeps` and `internal/battencli#battenCommitStatusDeps`, and via `pushOwedRunCommits`.
   - For such a call, `<hold>^` names a commit of the pair's records repository, not the prime's. The push would fail, or would hold back the prime's records branch.
   - Card 9's rows pin only a pair `l`. A prime-`l` row with a held run is missing.

3. MEDIUM — D5 criterion "later dobby ticks start no pusher for it" conflicts with card 11.
   - Card 11 has `starter.start` spawn a settle-only pusher while the refused record stands.
   - Its `TestParentPush` row asserts "`starter.start` spawning only a settle-only pusher", which is the opposite of the criterion.
   - D5's "keeps later ticks from starting the pusher" is not carried.

4. MEDIUM — D6 wait record, card 2: the record is removed when each wait returns.
   - While the re-step itself runs, no record stands, and the child's status reads failed. The run watch (card 3) and `failedHalt`'s held-lock read then see a stop between waits.
   - D6 wants the run read as waiting for the whole wait.
   - D3's "run kept a wait record for the threshold" needs the record, or its start time, to persist across re-steps.
   - `TestGoDriver_Settle` asserts the record is gone after each wait. No row pins the watch or the summary across the step in between.

5. MEDIUM — D2, card 5: "the pusher's pushes run `pushClassified` with no transport backoff of their own" is carried by no card.
   - Card 5 changes no push path, neither `PushPairAnchored` nor `RunPush`, to drop the transport backoff.
   - `TestRun`'s row "one push per interval after a transport failure" uses a fake `Push`, so it cannot fail when the real push retries internally.

6. MEDIUM — D7 criterion "the pusher stops at once with one record and one notice on an authentication refusal".
   - Card 5's row asserts one push and the refusal returned, over a fake `Push`.
   - Nothing asserts one failed-push record and one notice.

7. MEDIUM — D1 criterion "a landing onto a beam parent records its landing entry on the beam pair's target and creates no landing target".
   - No row pins it. Card 12 tests only that the `LandLocally` seam is called.
   - Card 11 lists `runpusher#RecordLanding` with no test, and `loomcli#landLocally` (card 12) has none either.

8. MEDIUM — the beam parent path of card 11 has no row:
   - `PushTip` as the pair's own push;
   - the owner reads through the beam pair's run-scope reconcile;
   - "a beam parent's push carries every landing, so it is held back whole".

   Every `TestParentPush` row is prime-shaped.

9. MEDIUM — card 11 branches with no row:
   - a pushed tip carrying a first-parent landing commit no landing records is refused (D5's "same refusal stands for any parent push whose tip carries a landing commit no entry records");
   - a dropped run this machine owns, `ResetToOrigin`, resets the local parent.

   On operands: D5 names `git reset --keep origin/<parent>`, but card 11 uses `fabricengine.ArchiveParentMerge`. That function also pushes an archive tag to origin and resets, so it is not the operand the record names.

10. MEDIUM — the "ahead by only owed tips" check, cards 12 and 16: both test a near operand.
    - Card 12 checks that the head equals an owed tip. Card 16's `primeAheadByOwedTips` checks that the tip is "one of them, or an ancestor of the newest".
    - D5 says "ahead of origin by only tips its parent target records as owed". An unrecorded commit between origin and an owed tip passes both checks.
    - No row has a prime or parent ahead by an owed tip plus an unrecorded commit.

11. MEDIUM — D13, every teardown refuses "before any mutation".
    - Card 19 places the refusal in `Teardown.Run` before `EndSession` and in `RemovePair`.
    - Batten cancel's `Teardown.EndForCancel` ends owned processes, strands and the session before its `RemovePair` refusal, so a cancel with a standing entry mutates first.
    - `HandOver` reached from `PushRun`'s takeover path calls `td.RemovePair` and would now fail with `ErrLandingStands` while an entry stands.
    - No row covers either route.

12. MEDIUM — D3 and D11: card 6's changes to the dobby tick have no row.
    - The tick now also restarts the board and landing pushers and runs `PruneLedger`, but card 6 relies on existing tests that pin only the prime and pair restarts.
    - `PruneLedger`, D3's "a dobby tick that finds the target's worktree gone" drops the entry, has no test.

13. MEDIUM — D3 criterion, a refused target "keeps the bar's minutes counting".
    - Card 21's `TestSyncStatus` has no refused-entry row.

14. MEDIUM — D1 and #238 "starts or wakes": `pusherCmd --push-now` sets the mark before the lock, and a second process exits at a held lock.
    - No row pins either.
    - Card 5's production `Wait`, which ends early on an `internal/fswatch` push-now event, is untested. Every row uses a pusher that finds the mark at its start.

15. LOW — D6 criterion "red-first test in `internal/sheddriver` … tier1": card 2 puts the rows in tier0 `TestDecideDrive` and `TestGoDriver_Settle`. The record names tier1.

16. LOW — D6 halt-note criterion names `git-transport` or `github-api`. Card 3's `TestLoomPostRun_HaltNotes` rows cover `git-transport` only.

17. LOW — D13 criterion: `lym fabric remove` should leave origin's board without `done` and origin's records without the held commit.
    - Card 19's rows assert only "records no call" over fakes, and an envelope row in `TestRemover`.
    - Origin's state is not asserted.

18. LOW — D8 criterion "starts … the run": card 17 tests only `pushOwedForCatchUp` returning nil, not `CatchUp` starting the run.

19. LOW — branches with no row:
    - `ReadDriveWait`'s failed read (card 3);
    - a zero `Timing` reading as the defaults, and a level pass removing the entry versus keeping it with landings (card 5);
    - the board target's refused-push condition (card 6);
    - `RecordsHold` already on origin, push skipped (card 9);
    - `PushPrimeCodeTip`'s owed transport failure with no record (card 10);
    - a parent-owner read transport failure as `git-transport` (card 12);
    - a nil `ParentPushState` (card 13);
    - a prime ahead by a non-owed commit judged as today (card 16);
    - a failed summary post retried next tick (card 20);
    - `sync: ?` on a failed read (card 21);
    - the landing target pulls the Bolt before its owner reads (card 11).

20. LOW — D5 "a landing that needs GitHub checks before it takes the parent lock": card 13 runs the lock-free check only when a pull-request record names one. A trail landing with no record is checked only under the lock.

No further tool calls were made.
