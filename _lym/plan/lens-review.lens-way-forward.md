```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: I checked each refusal, stop and recovery step the cards add against whether the next step it names can succeed from the state it leaves.

**Findings**

1. **MEDIUM — Card 2 (D4): a failed record move or removal skips the render and the sync of a write already saved.** The card puts `followDecisionRecords` after the save and `afterSave` but before the render, and returns its error. In `boardCriticalSection`, an error at that point returns before `RenderToDisk` and `spawnSync` (internal/boardengine/board.go:156-170). So a rename or remove that has already saved `board.json` is neither rendered nor pushed until some later write. Card 2's "A rename whose target `decision-<new>.md` already exists as a directory" row would ship that state. The step should still render and spawn the sync, then report the file error. Alternatively it should run as a best-effort step that logs.

2. **MEDIUM — Card 2 (D4): a failed record removal gives no way forward, and the obvious retry is refused.** The removal branch says "any other failure is returned", with no message. The entry is already saved as removed. Re-running `lym board remove` meets `task not found` (internal/boardengine/store.go:598). No later write retries the removal either, because the slug is no longer in `loaded`. The error should name what happened (the entry is removed, the file `decision-<slug>.md` remains) and what to do (remove the file by hand, or leave it, since `DecisionRecord` requires the flag).

3. **MEDIUM — Card 7 (D6): on the accepted-round path, a resume after a failed store does not store again.** Card 7 says "a resume re-settles and stores again". But `acceptRound` settles the accept (`settleCirclingAccept`) before `approveAndCommit` (internal/shedadapters/bouncer.go:1276-1281). After a failed approve or commit, the resumed call clears and archives the generation without calling Approve again. That is pinned by `TestBouncer_Circling_AcceptWithFailedCommitResumesByClearingWithoutApprove` (internal/shedadapters/bouncer_circling_test.go:240-262, "want no second Approve on the resume"). So after a failed store on an accept, the resume reopens a full review round instead of storing. That contradicts the D6 criterion "a failing store halts the run at Discussion-Bouncer, and a resume stores it". No planned row reaches the accept path with a failing `ApproveDiscussion`, so the card's claim is untested.

4. **MEDIUM — Card 7 against Card 1 (D2, D6): a done entry blocks Discussion-Bouncer for good.** `ApproveDiscussion` returns every store error, and `StoreDecisionRecord` refuses a done entry with `ErrRecordEntryDone`. If a run at Discussion-Bouncer has an entry marked done (for example a hand `set-status`), every settle fails. Every resume fails the same way. The refusal's text only says "a landed entry keeps the record it last stored", which names no step that lets the run proceed. The seam should treat `ErrRecordEntryDone` as a no-op that it logs. Otherwise the refusal must name a step that progresses, such as clearing the status.

5. **LOW — Card 8 (D8): a missing entry gives two ways forward, and the second cannot succeed until the first is done.** When the store fails because the entry is absent, the board error already says "restore the entry with `lym board upsert`, then re-run the step that stored". `copyRecordToBoard` then appends `decisionWayStore`. Running `lym loom decision store` first meets the same absent-slug refusal.

6. **LOW — Card 9 (D9): an empty parent name renders a broken refusal.** The text is "ask %s to run …", and the card covers only the case where the parent cannot be resolved. `add` handles `parent.Name == ""` separately (internal/loomcli/decision.go:672-673). As written, `store` would print "ask  to run".

7. **LOW — Card 9: the missing-committed-record refusal omits the clause about who runs `lym fabric commit`.** It names `lym fabric commit` without the clause `decisionWayFabricCommit` carries (internal/loomcli/decision.go:102): "a session lym refuses the verb from reports status: FAILED and the orch runs it". A task session following the card's text is refused with nothing further to do.

8. **LOW — Card 3 (D1): the way forward does not name the verb.** It says "set design_prefix in board.yaml" but not `lym config board --set design_prefix=design-` (the example form at internal/configcli/configcli.go:477). An agent session may not edit the Bolt checkout's file by hand. `configcli` does not go through `boardengine.LoadConfig`, so that verb still works while the board refuses.

9. **LOW — Card 9 (D9): the not-past refusal assumes `lym loom resume` can settle Discussion-Review.** A run halted in Discussion-Review on a CIRCLING or decision escalation is waiting on a parent decision (`lym loom circling …`), not a resume. Resuming there does not settle the segment, so the named next step does not progress in that state.

**Refusals that progress:** Card 1's absent-slug and payload refusals, Card 5's four `--record` refusals, Card 8's failed-commit refusal (`lym fabric commit`, then `decision store`), and Card 9's caller, no-approval-row, done-entry and retry refusals all name a next step that works from the state they leave.

**Tool calls**
- Read internal/shedadapters/bouncer_circling_test.go:230-300, to check whether a resume after a failed seam on the accept path re-runs Approve; my context did not show the accept-resume behaviour.
- Grepped loomcli for how run ids are set, and configcli, hubreconcile and configsync for `boardengine.LoadConfig`, to check that the slug the seam stores under matches `SetRunStatus`'s and that the D1 refusal's way forward is reachable.
- Read internal/loomcli/arm.go:80-100 and cli.go:255-268, to confirm the run id comes from the positional slug.
