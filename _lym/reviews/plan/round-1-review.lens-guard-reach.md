```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch or test row the cards add: can its state occur, and does the guarded site recur on a path the card leaves unguarded?

**Per guard, reachable or not**

- Card 1, the `checkTaskSetting` autonomy refusal (`"yes"`, `"on"`) in the lookup, `LoadConfig`, `CheckLocalValues` and `TestPrepareRunSettings`: reachable.
- Card 3, `Autonomy` reads false on a refused value and on a non-git location: reachable. `dotgit.ListWorktrees` fails on a non-git directory, and the lookup refuses a beam file holding `"yes"`.
- Card 5, `ClearParentAsk`: the refusals of a missing number and of an open ask cannot occur in production. Its only caller, card 6's `reaskParent`, calls it only after `endedUnanswered` found the ask ended.
- Card 6, the ended-ask halt in `DriftVerify.judge`: reachable through `agreement` passing on timeout, on not-sent and on no-parent. The same site recurs unguarded in `passCommand`.
- Card 6, the `reaskParent` resume and the `Cleared` branch in `agreementGate`: reachable from `judge` only. The same site recurs unguarded in `passCommand`.
- Card 10, the nil-seam wiring error: a fail-loud wiring guard, so it needs no reaching test.
- Card 10, the `AskPRReview` error branch: reachable.
- Card 11, `judge` Open with `Continue` and a standing other than satisfied: reachable (a commit after `pr-continue`, then a gate). The session stop that comes before it recurs unguarded in `sessionStopWanted`.
- Card 11, Closed with autonomy, and Finalize's closed halt under autonomy: reachable. Both reopen sites (`finalize_pr.go:361`, `:476`) go through `landingForDecision`'s decision.
- Card 12, refusals 1 to 4 of `parent-review ask`: each reachable. `preRun` only needs a loom seed recipe, so no upstream check rejects them.
- Card 12, the PR-Review branch of `parentAnswerRefusalText`: reachable for the past-deadline and ended cases. The "replaced" test row reaches it only when the newer ask has ended.
- Card 13, the closed refusal and cases 2 to 4: reachable.
- Card 14, `prReviewResettingGuard` reset on a run at or past PR-Review: reachable.
- Card 14, its "run halted before PR-Review" branch: not reachable through `Goto` or `Relaunch`.
- Card 14, the reset inside `Relaunch`: runs before `Admit`, which can still refuse.
- Card 15, `noticeDrain`: reachable on the wakes that find no land request.
- Card 16, the unmarked-reply drop: reachable.

**Findings**

1. **BLOCKING** (decision), card 6 `06-drift-autonomy.md`: under autonomy, an ended judge ask still lets the rulings stand in diff-command mode.
   - `passCommand` runs the same judge, `runAgent` with the parent agreement entry from `tableWithParent`, then calls `storeCaseRulings` with no `endedUnanswered` check.
   - Card 6 adds the halt and the `reaskParent` resume only in `DriftVerify.judge`.
   - D3's halt "at Drift-Verify" is therefore missing for command-mode runs.
   - Evidence: `internal/driftengine/verify.go:1072-1076` (passCommand → runAgent → storeCaseRulings) and `:476-480`.

2. **MEDIUM** (design), card 11 `11-pr-review-ending-autonomy.md`: the mail telling the session to ask again goes out while the session is stopped.
   - Under autonomy, `Continue` set on a clean, gated head makes `sessionStopWanted` true whatever the standing, so the row stops the session before `judge`.
   - `judge` then clears `Continue` and mails `rec.StrandName`, the stopped strand.
   - Only after that does `resumeSession`, through `ensureSession`, start the session again, possibly under a new strand name.
   - Each such wake cycles the session, and the "ask again" mail can miss it.
   - Card 11 leaves `sessionStopWanted`'s `Continue` case ungated by the standing.
   - Evidence: `internal/landingshed/prreview.go:660-673` (stop → judge → resume), `:690-691`; `prsession.go:226-236`.

3. **MEDIUM** (design), card 14 `14-goto-clears-pr-review.md`: the reset can clear and commit the record even when no move happens.
   - `Relaunch` runs the guard at `goto.go:244`, then `Admit` at `:248`, which can return an error or `inEffect`.
   - `arm.go:478` wires `spec.GotoGuard` through `gotoGuardEndingFixers`.
   - So `clearPRReviewOnGoto` clears and commits the pull request record's autonomy and session while the run stays where it was.
   - In `Goto`, a failed status write after the guard does the same.
   - Evidence: `internal/shedengine/goto.go:117-139`, `:244-248`; `internal/loomcli/arm.go:478`.

4. **LOW** (design), card 14: the `atOrPastPRReview` false branch and its test row ("run halted before PR-Review resets nothing") cannot occur.
   - `CheckGotoTarget` runs before `checkGuard` and refuses any target past the current row.
   - So a target of PR-Review always means the run is at or past PR-Review.
   - Evidence: `internal/shedengine/goto.go:114-119`, `CheckGotoTarget` at `:146-160`.

5. **LOW** (consistency), card 12 `12-parent-review-ask-verb.md`: the test row "an answer to an earlier PR-Review ask that a newer one replaced is refused the same way" does not say the newer ask has ended.
   - With the newer ask still open, `ParentAnswerNotLatest` takes the existing "awaits an answer" text, not the card's PR-Review way forward.
   - A newer ask at a new head leaves the older one open, since `RecordParentAsk` ends no earlier ask.
   - Evidence: `internal/loomcli/parentreview.go:165-168`; `internal/burlerengine/parentreview.go:200-215`.

6. **LOW** (design), card 5 `05-parent-ask-clear.md`: `ClearParentAsk`'s refusals of an open ask and a missing number guard states its one caller never produces.
   - The card's test rows reach them only by calling the function directly.
   - Evidence: card 6's `reaskParent` calls it only after `endedUnanswered`; the `ParentAsk` states are at `internal/burlerengine/parentreview.go:44-46`.

**Tool calls**

1. Grep of every `Continue` read and write and every `Reopen(` site, plus `sessionStopWanted`: needed to check whether card 11's guard recurs on other paths, which the manifest lacked.
2. Grep of the `gotoGuardEndingFixers` callers and `Relaunch`'s guard and `Admit` order: card 14's reach goes through `Relaunch`, which the manifest lacked.
3. Read of `BeamLandBlockers` and `Empty`: needed to check that card 15's `WorkRemains` condition can become false.
4. Read of PR-Review `wake` (`prreview.go:605-680`): needed for the stop → judge → resume order around card 11's branch.
5. Read of `stopLiveSession` and `resumeSession`: needed to see what happens to the session after card 11's mail.
