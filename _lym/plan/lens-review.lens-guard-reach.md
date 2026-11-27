```yaml
lens: guard-reach
outcome: reported
```

Twelve findings: three BLOCKING, seven MEDIUM and two LOW.

1. **BLOCKING — D5 hold and D13 dropped landing (cards 9, 11, 19).** Once a landing is dropped, nothing keeps the held records commit off origin any more.
   - Card 9's hold lasts only while the landing entry stands: `PushRun` sets `RecordsHold` from `pushledger.HeldRecordsCommit`, and the overview's Shared Decision says the hold lasts "exactly as long as the entry".
   - Card 11 step 3 drops a landing by removing its entry first.
   - Card 11 then hands an elsewhere-owned run over through `pairteardown.HandOverOwnedElsewhere`. That function calls `pushOwedRunCommits`, which calls `PushRun` (`internal/pairteardown/handover.go:213,229,244`). With the entry gone, `PushRun` sees no hold and pushes the held commit and everything after it.
   - A dropped run this machine owns keeps its pair. Its own pusher (cards 5 and 8) then pushes the held commit the same way.
   - Card 11's claim that the hand-over's "pushes stop short of the held commit (card 9)" therefore cannot hold. This breaks D5 ("the entry's pair hold outlives it … the hold goes with the pair"), D13's dropped-landing sentence, and the D5 criterion that no origin branch receives the held commit.

2. **BLOCKING — D5 hold (card 9).** `PushRun` would set `RecordsHold` on pushes of the prime's records too.
   - The card has `PushRun` read `HeldRecordsCommit` by slug alone. `PushRun` also pushes the prime's records under a run slug: `pushOwedForCatchUp` (`internal/battencli/catchup.go:267`), batten's commit-status `Push` (`internal/battencli/commitstatus.go:183`), cancel's `PushRecords` (`internal/battencli/cancelcmd.go:365`) and `pushOwedRunCommits` (`internal/pairteardown/handover.go:229`).
   - In each of those, `PushPairAnchored` would push the prime's records branch only up to `<hold>^`. That is a commit on the pair's records branch, not on the prime's.
   - D5 says batten's prime records are not held. The hold must apply only when `l` is the run's own pair.

3. **BLOCKING — PATTERN-owner-machine and D5 carried-run checks (cards 8, 11).** A beam parent's landing merge can reach origin through paths that skip the carried-run owner reads.
   - Card 8 makes `RunCommitsAhead` count every code commit ahead of origin. A landing merge on the beam's code branch therefore makes the beam pair owed, so the beam pair's ordinary push sends it.
   - That ordinary push goes through `PushRun` from the pair pusher (card 5), the catch-up (`internal/battencli/catchup.go:274`) and `lym fabric push` in the beam worktree.
   - None of these runs card 11's reads of each carried run's owner, or its refusal of a landing commit no entry records. Card 11 gates only `settleParent`'s own `PushTip`.
   - So a run adopted elsewhere, landed on a beam, still reaches origin. D5 says the push goes "only while no other machine owns the parent or any run the push carries".

4. **MEDIUM — D3 and D5 refusal reads (cards 6, 11).** The landing and board targets read the wrong refused-push record.
   - `newStarter.refusedPushStands` and `NewDeps`' `Conditions` both call `fabricengine.RefusedPushStands(t.Location)`. For the prime that function skips the code branch (`internal/fabricengine/pusherreads.go:162-167`).
   - Card 6 builds the landing and board targets over the prime and does not say their starter or `Conditions` change. Card 11 adds `TargetRefused` but wires it only into `RecordLanding` and the summary.
   - Effect on the landing target: a `held_back` record of the prime's code branch never stops it, and a refusal of the prime's records stops it wrongly. Every tick would start a full pusher, which pulls the Bolt and refuses again. That breaks the D5 criterion "later dobby ticks start no pusher for it".
   - Effect on the board target: it is blocked by a refusal of the prime's records branch and not by a refusal of its own Bolt branch.

5. **MEDIUM — settling a beam parent (cards 5, 6, 11).** A beam pair's landings can be left unsettled.
   - A beam pair is a records target whose `Ahead` is `RunCommitsAhead`. Once its code side has gone out through the ordinary push, it reads level and the pass exits. Card 5 says the entry is kept because it holds landings.
   - No card adds "holds landings" to that target's `Ahead` or to `StartIfAhead`. Only the landing target's `Ahead` is "a landing entry stands" (card 6).
   - If the pass exits before `settleParent` sees origin holding the tip, nothing restarts it. The landing's `done` write, its pair hold and its child pair's teardown then wait forever.
   - Card 11's settle-only start fires only while a refused-push record stands.

6. **MEDIUM — D1 red-first criterion for issue #238 (card 7).** `TestSpawnStatusPush`'s red-first half cannot fail as described.
   - The test records `spawnDetachedPushFn` instead of running the child.
   - The merge base's "owed failure writes a failed-push record and mails the orch" happens inside the `fabric push` child. A recorder never runs that child, so the merge-base row can only compare command lines.

7. **MEDIUM — D13 teardown refusal (card 19).** A takeover hand-over hits card 19's own refusal.
   - When a rejected push meets a takeover entry, `PushRun` calls `HandOver` (`internal/pairteardown/handover.go:106,116`).
   - `HandOver` first ends the run's processes and archives its tips (`HandOverTips`), then calls `td.RemovePair` (`handover.go:191`).
   - With the run's landing entry standing (no drop happened), the new `ErrLandingStands` refuses after those steps have already run. `HandOver` returns a plain error rather than `ErrHandedOver`, so the hand-over is half done and repeats on every rejected push.
   - The plan names only the pusher's drop path as passing the check.

8. **MEDIUM — D5 owed-landing check before the parent lock (card 13).** The criterion "stops as git-transport before taking the parent lock" is met only for a landing that already has a pull-request record.
   - The pre-lock check fires only when the run's or the beam's record names a pull request.
   - A trail landing, or any rule-yes landing with no record, is caught only by the second check, under the lock. The card's trail row asserts exactly that.

9. **MEDIUM — D5 refusal of an unrecorded landing commit (card 11 step 4).** The guard depends on a fact the card never pins down.
   - "A pushed tip carrying a first-parent landing commit no landing records is refused" requires telling a landing commit from any other parent commit. No card says how.
   - On the prime it can only occur when a Finalize crashed between its merge and `LandLocally` and another landing then stacked on top. Card 12's resume re-records that case, so the window is narrow.
   - No `TestParentPush` row reaches it.

10. **MEDIUM — removal routes refuse before any mutation (card 19).** Cancel's route mutates before the refusal fires.
    - `EndForCancel` ends the owned processes and strands (`internal/pairteardown/cancel.go:64`) before `RemovePair` (`internal/battencli/cancel.go:843`).
    - `lym batten cancel` of a run at Land whose landing entry stands therefore ends processes, then meets `ErrLandingStands`. That breaks D13's "refuses before any mutation".
    - Card 19 guards only `Teardown.Run`, before `EndSession`, and `RemovePair`.

11. **LOW — owner check on an unreachable origin (card 12).** The branch "a parent-owner read that fails on a transport error" returns `git-transport` cannot be reached.
    - `Deps.ParentOwner` reads the local board that the reconcile left, per its doc in `internal/landingshed/deps.go:159`. It makes no network call that could fail on transport.

12. **LOW — the run watch's wait state (card 3).** A short race can still mail a stop line.
    - The driver writes the `DriveWait` record in `settleStop` after the step has already persisted its failed status.
    - A dobby tick in that gap reads the child as failed with no wait record and owes a stop line, so a mail can still go out at the start of a wait.

Tool calls:
- I grepped `internal/battencli` and `internal/battenshed` for `sheddriver`, to confirm batten's Land row runs under the Go driver so card 16's transient stop is waited out. My context lacked who drives batten.
- I grepped the `RemovePair`, `RemoveHandedOverPair`, `RemoveOrphanPair` and `EndForCancel` call sites, to check whether every removal route meets card 19's refusal and in what order. My context lacked the removal call graph.
