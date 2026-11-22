```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** For each function or branch the diff changes, I ran one concrete input and checked the result.

**List (function or branch: case run, verdict)**
- `hubgeom.Autonomy`: slug "" reads the prime chain, correct. The prime's own name reads the prime chain, correct. A beam slug starts at the beam, correct. A non-git location reads false with one warning, correct. A refused value reads false with one warning, correct. `worktrees[0]` is unguarded, but `ListWorktrees` always returns the main worktree, so I left it.
- `dobbyengine.checkTaskSetting` and `LookupAutonomy`: bool `true` gives "true"; string `"yes"` and `"on"` are refused naming the key; `false` gives false. Correct.
- `Finalize.configChangeNotice`: an autonomy-only change names nothing and queues no notice; autonomy plus `max_runs` names only `max_runs`. Correct.
- `Finalize.landingForDecision`: a closed pull request with recorded autonomy halts whatever `ReviewEnded` and `MergeAtOnce` say, and reopens nothing. Correct.
- `PRReview.judge`:
  - Autonomous, open, `Continue`, gated, standing satisfied: ends the review. Correct.
  - Standing ask-again: clears `Continue` and returns the ask-verb mail. Correct.
  - Standing awaiting: returns the await-verb mail. Correct.
  - Autonomous and closed with `Continue`: halts, with the standing never read. Correct.
  - Approval at the gated head: ends with no standing read. Correct.
- `PRReview.wake` and `mailStayOpen`: the mail goes out after the resume, to the strand name the record holds then. Correct.
- `ensureSession`: an unrecorded autonomy is recorded once, and a recorded value outlives a layer change. Correct.
- `prContinueUnderAutonomy`:
  - Closed: refused naming the row's halt. Correct.
  - Satisfied at the head: sets `Continue` and clears the mailed head. Correct.
  - Not satisfied with a mailed head: clears it and answers ok with `continue` false. Correct.
  - Not satisfied with none: refused. Correct.
  - `parentReviewAskVerb`: an unanswered ask at the head is refused naming await, and a non-session caller is refused. Correct.
- `ReadPRReviewStanding`: another tag is never read as a PR-Review ask. Done with findings gives ask-again. Open past its deadline gives ask-again. Cleared gives ask-again. Correct.
- `PRReviewAsker.Ask`: resolve error or no strand records no-parent and posts nothing. A failed post ends the ask no-parent. Correct.
- `DriftVerify.runJudgeSession`, `askToRepeat` and `reaskParent`:
  - Timeout, not-sent and no-parent all halt under autonomy, and a resume clears the ended ask and re-asks.
  - A re-ask answered with findings falls through to a judge resume; a re-ask that ends again halts again.
  - Clearing an ask and then dying before the new ask is sent leaves the latest ask ended and cleared. The next resume then skips `reaskParent` and spawns a judge. The agreement gate then sends the new ask, so nothing is lost; the only effect is a judge spawn the re-ask path meant to avoid.
- `agreementGate` with a cleared held ask: sets round to Round+1 and sends a new ask. Correct.
- `noticeDrain`:
  - Work remains: removes the marker. Correct.
  - Holds-commits false: no post. Correct.
  - A ready record stands: no post. Correct.
  - A standing marker: no post. Correct.
  - A post that reached no orch: writes no marker. Correct.
  - A seam error: decides nothing. Correct.
- `loomengine.ApplyAutonomy` and `driftengine.ApplyAutonomy`: forced keys are right. `AutonomyForcedSetting` covers all five keys across the two modules. `LoadConfig` refuses `pr_review_parent_timeout_min` below 1. Correct.
- `autonomySettingLookup`: reads autonomy per forced call; an unforced key passes through. Correct.
- `slackcli.replyHook`: filters only when autonomy is on and the reply lacks the marker, after the presence check. Correct.
- `WithLaunchFacts`: the line is appended after the presence line and one-line. Correct.
- `NotifyParentHalt` and `NotifyRunFromHalt`: both append `; autonomy: &lt;v&gt;` read at build time. Correct.
- `ClearReviewSession` and `clearPRReviewOnGoto`: a missing record is a no-op; a stop error clears nothing. Correct, except for F1 below.

**Findings**

F1. MEDIUM, design. Location: `internal/pullrequest/record.go` (`ClearReviewSession`), with `internal/landingshed/prreview.go:384`.
- `ClearReviewSession` clears `Autonomy` and the session fields but keeps `ParentReviewAsked`. Only `settleSendBack` (`prreview.go:124`) resets it.
- Take a record with `ParentReviewAsked` true that is then moved by `lym loom goto` onto PR-Review. The re-entry records autonomy true anew and starts a new session. `askParentReview` returns at once because `rec.ParentReviewAsked` is set.
- The row therefore never sends the first PR-Review ask. D4 says a goto re-entry "records the value anew and starts a new session", and that a re-entered review "sends a new first ask".
- The autonomous prompt only says to ask after each set of fixes, so a session with nothing to fix has no ask. `pr-continue` refuses as "ask your parent again", and the agent has no instruction to take that step.
- Evidence: `internal/landingshed/prreview.go:384` and `record.go` `ClearReviewSession`.
- Fix: clear `ParentReviewAsked` in `ClearReviewSession` and add a row to `TestClearReviewSession` for it.

F2. LOW, design. Location: `internal/loomcli/prreviewgoto.go` (`prReviewClearFailure`) with `internal/shedengine/goto.go`.
- On a failed clear, the error says to "run lym loom goto --to PR-Review again to repeat the clear". The move is already written, so the run now stands at PR-Review.
- `Goto` accepts a target at or before the current row. In `Relaunch`, a target already in effect runs no goto and so never calls `Admitted`.
- The retry looks like it may not repeat the clear on the `Relaunch` path. I could not confirm that `Goto` itself re-runs `Admitted` on a same-row target. A `Goto` onto the current row would be refused or never reach the hook.
- Evidence: `goto.go` lines ~202 and ~222 (`Relaunch` skips the hook when the target is in effect).
- Fix: find a way forward that works from the written state, or verify by test that a repeat goto onto the current row runs `Admitted`.

**Tool calls**
1. `grep ParentReviewAsked` in `internal/*.go` and `grep` in `shedengine/goto.go`: the manifest did not show who clears `ParentReviewAsked`, or whether a same-row goto re-runs `Admitted`.
