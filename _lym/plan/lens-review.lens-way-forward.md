```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop or recovery step the cards add, I checked the next step it names against the state it leaves. Only defects follow.

1. **BLOCKING — Card 9, `refuseSourceMarkerChange` / `ErrMarkerChange`: both named ways forward can never clear the refusal.**
   - The refusal fires when any commit between the merge base and the source tip changes `.lym-anchor` without `AnchorChangeTrailerKey`.
   - Way forward (a), revert the marker edit on the source branch: this leaves the bad commit in that range and adds a second untrailered marker change, the revert itself.
   - Way forward (b), redo the anchor change through `lym fabric anchor`: this leaves the original untrailered commit in the range too.
   - Either way the next merge refuses again, forever.
   - Evidence: Card 9's Intent, step 1, defines the scan per commit and names that way forward.
   - Fix: judge the net change, meaning the marker at the source tip against the marker at the merge base, with the change admitted only when the newest commit that set it carries the trailer. Or name a way forward that removes the commit.

2. **BLOCKING — Card 8, integrate kind refused in one repository: the way forward loops back into the same stop.**
   - `BeginAnchorChange` picks `MoveKindIntegrate` exactly when `upstreamCarriesAnotherAnchor` is true (`internal/fabricengine/anchorchange.go:181-187`).
   - Card 8 refuses that kind with "pull the pair level with origin, then run the anchor change again".
   - But the pull stops on that same differing anchor through `weftAnchorStop`. Its way forward is "run `lym fabric anchor <slug> --set <want>` …, which integrates the other marker" (`anchorstop.go:171`), and that is refused again.
   - So the run waits on origin's anchor move with no route out. Card 8's row `OriginAtAnotherAnchorStops` asserts the stop and never a recovery.
   - Fix: give the one-repository integrate a real path, such as resetting the records paths to origin's anchor by replaying the records-only commits (card 13). Or name a way forward that progresses from the stop, and add a row that reaches it.

3. **BLOCKING — Cards 12, 13 and 14, failed-push records in one repository name an empty directory and an empty run.**
   - `pairedCodeWorktree` returns `""` when the records worktree has no `-weft` suffix (`pushclassified.go:475-481`). That is every one-repository worktree once card 3 makes `RecordsWorktree` the worktree itself.
   - Every records-side failed-push record (`pushclassified.go:354`, `pushanchored.go:309`, `:373`) therefore carries `CodeWorktree: ""`.
   - `pushStep` then renders "`lym fabric push` in " with no directory (`failedpush.go:348-349`, through `lymVerbDir("")`), so every way forward and creel notice points nowhere.
   - `pairSlug()` returns `""`, so the notice carries no run and the pair's teardown never drops it (`failedpush.go:218-227`).
   - Card 12 says the record "names the branch once, as the records side, so `PushOwed` and the act rule read it as today", but no card edits `pairedCodeWorktree` or `pushStep`.
   - Fix: in a one-repository worktree, set `CodeWorktree` to the worktree itself. Add a `TestFailedPushRecord` row and a `TestPairPush_OneRepository` assertion on the way forward's directory.

4. **BLOCKING — Card 13, the replay stop for a local code commit: wrong way forward, and the run waits with no notice.**
   - The stop arises only when origin diverged and a local code commit is unpushed. Card 13 names "the commits and `git push`", and a plain `git push` of a diverged branch is rejected as non-fast-forward. The settling step is today's `git merge origin/<branch>` then push (`codeDivergedWayForward`, `failedpush.go:406-413`).
   - `retryWeftPush` records the stop as a deferred push. `recordPushFailure` sends no notice for `PushFailureDeferred` (`failedpush.go:170`), and `IsPushOwed` reads deferred as owed.
   - So the transition seam retries forever, behind the gate, and nothing ever tells the orch or the operator.
   - D7 says "stops as owed, naming the commits"; it does not say silently.
   - Fix: use a class that notifies, such as `not_fast_forward` with `Commits` set, or a new owed-and-notified class. Name the merge-then-push way forward, and assert the notice in `TestReplay_OneRepository`'s `LocalCodeCommitRewritesNothingAndStopsNamingIt` row.

5. **MEDIUM — Card 14, guard refusal recorded as `protected` once a `protected` record exists: the way forward goes stale and the record never clears.**
   - A record is cleared only by a landed push (`pushJob.landed`). The guard pushes nothing while operator commits are unpushed.
   - So after the operator allows direct pushes, every later guard refusal is still recorded `protected`. The seam then completes past it (card 11), and the notice keeps saying "allow direct pushes", which the operator already did, instead of naming the unpushed commits and `git push`.
   - Fix: record `protected` only while the latest push attempt of the branch was refused as protected. Or let the guard's refusal name the operator's commits as well. Add a row covering "protected record exists, protection lifted, operator commit unpushed".

6. **MEDIUM — Card 7 `refuseOffPrimeBranch` with card 11's seam: the refusal halts the run, where D7 says the run stalls.**
   - Card 11's `failureDisposition` waits only when the re-probe names an operation. An off-branch refusal has no operation, so the transition returns the error and the run halts.
   - The refusal's way forward is "switch the prime back … then run the step again", but it does not say how a halted run resumes.
   - D7's Bound says "prime runs stall until the switch back".
   - Fix: let the seam wait on the off-branch and missing-key refusals as it waits on an operation, with a row in `TestNew_MergeOperationWaits`. Or name the resume step in the way forward.

7. **MEDIUM — Card 7 `refuseOffPrimeBranch` gate: it can bind two-repository primes.**
   - The card applies it "when `worktree` is the one repository's main worktree (its gitdir equals its common dir)".
   - In a two-repository hub, the records worktree passed to `CommitRecordsPaths` and `commitWeftLocked` is the weft prime. That is the main worktree of the records repository, so gitdir equals common dir there too.
   - Such a prime has no `lym.primeBranch`, so every two-repository prime records commit would refuse, naming a key whose way forward misdirects the operator.
   - Fix: gate the check on `lymcwd.OneRepository` explicitly. Add a `TwoRepositoryPrimeIsNotRefused` row to `TestRecordsCommit_OneRepository`, or assert it in an existing two-repository commit test.

8. **MEDIUM — Card 9, step-3 refusal after step 2: the refusal leaves a parked unsettled merge, and the card names no way forward from it.**
   - The refusal comes after `MergeStartNoFastForward` and the saved merge record. Today's dirty refusal happens before anything is merged, and its implied next step is to clean up and merge again.
   - Here a retry hits the already-in-progress guard instead.
   - The card states neither a self-abort, as `Fabric.Merge`'s conflict path resets the sides, nor the way forward `lym fabric merge --abort`.
   - Fix: state which one happens, abort or park, and name `lym fabric merge --abort`, then merge again. Assert the leftover state in the `RecordsWriteBetweenStepTwoAndThreeRefuses` row.

9. **LOW — Card 10, refusal of other merge states (a foreign merge with records unlike HEAD, a cherry-pick, a revert or a rebase) to non-seam callers.**
   - The card says the refusal is "refused with the operation named", with no way forward.
   - `lym fabric commit`, `lym fabric sync` and `webstercli`'s sync surface it to an operator. Today's `ErrForeignMergeState` names "conclude or abort it with plain git, then retry".
   - Fix: name that way forward in the refusal text.

10. **LOW — Card 14, way forward for `operator_unpushed`.**
    - "`git push` in the prime" fails when origin's prime branch moved, for example after another machine pushed. The way forward should say to merge or pull origin first when the push is rejected.

Tool calls:
- Read `commitAnchorMarker` (`anchormarker.go:51-87`), the `CodeWorktree` assignment sites, the integrate-kind call sites and the anchor-stop way-forward line, to check the clone re-run, the failed-push record fields and the card 8 loop. My context had only summaries of these.
- Read `pairedCodeWorktree` (`pushclassified.go:473-481`) and the integrate selection (`anchorchange.go:175-192`), to confirm that a non-suffixed worktree gets an empty `CodeWorktree` and when integrate is chosen.
