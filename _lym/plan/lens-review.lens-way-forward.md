```yaml
lens: way-forward
outcome: reported
```

Reviewed the plan's refusals, stops, resets and recovery steps for whether each named next step can succeed.

**Findings**

1. **BLOCKING: card 6, D3 resume re-ask.** Resuming the Drift-Verify halt starts a new judge session instead of re-asking the parent over the same rulings.
   - Card 6 halts after `runAgent` returns Done and leaves the judge session mark open.
   - On resume, `DriftVerify.judge` sees the open mark, runs `resumeJudgeSession` (where card 6 clears the ask), then calls `runAgent` again (`internal/driftengine/verify.go` judge body).
   - The judge chair has already ended, so `MultiLLMProducer.Call` finds no live chair. It calls `archiveStaleOutputs`, which archives `judgement.yaml`, then `seats.Run` starts a fresh judge (`internal/shedadapters/multillm.go:117-144`).
   - The resume therefore pays for a new LLM judge and may get different rulings. D3 says "the gate sends a new ask over the same rulings", and its acceptance criterion says "the resume sends a new ask over the same rulings".
   - Card 6's test row only asserts the ask is `Cleared` after the second call, so it never catches this.
   - Fix: when the open mark's latest ask is cleared and `judgement.yaml` is present, run the parent-agreement gate directly over the stored judgement, bypassing `runAgent`. The ask limit needs recomputing from the resume's `now`, since `judgeStartedAt` is set only in `runAgent`. Also assert in the test that no seat `Run` happens and that the new ask's digest equals the old one.

2. **BLOCKING: card 13, `pr-continue` refusal on a closed pull request.** The named next step cannot succeed for the agent that receives the refusal.
   - Card 13 names "reopen it on GitHub and run `lym loom resume`, or `lym batten cancel <slug>`".
   - The caller is the PR-Review session's agent, inside a live run. `lym loom resume` refuses a run that is not halted ("refuses everything else with a way forward", `internal/loomcli/resume.go:3`).
   - The agent cannot reopen on GitHub: PR-Review denies publishing, and PATTERN-github-auth applies.
   - `lym batten cancel` is the operator's call (D5 class 3).
   - The halt text it copies (`internal/landingshed/prreview.go:867`) is addressed to whoever settles a halted run, not to the session.
   - D4 asks that this way forward point to the row's closed-without-merge halt. Fix: tell the agent to stop and leave the pull request alone, because the PR-Review row halts on its next wake and the parent settles that halt. Assert that wording in the card 13 test row.

3. **MEDIUM: cards 10 and 12, a timed-out or unsent PR-Review ask is never re-asked.**
   - Card 12's answer refusal for an ended PR-Review ask tells the orch to "wait for the session's next `[pr-review request]` line".
   - That only happens if the session asks again. Card 10's autonomous `review_mode` text only says to ask after each set of fixes, wait with `await`, then run `pr-findings` or `pr-continue`.
   - It never says what to do when `await` returns an ask ended timeout or no-parent, which D4 requires ("the session asks again with `lym loom parent-review ask`").
   - The same gap applies when the row's first ask fails. Card 10 says a failed ask "is logged and the next entry asks again", but the row reaches `askParentReview` only on entry, before `wait`, so nothing asks again until a pause or resume.
   - Fix: card 10's text tells the agent to run `lym loom parent-review ask` when `await` reports an ended ask or no ask stands at its head. Card 10's prompt test asserts it.

4. **LOW: card 11, mail sent while the newest ask is awaiting.**
   - When the standing is `PRReviewAwaiting`, `judge` still mails "ask the parent again with `lym loom parent-review ask`".
   - Card 12's refusal 4 then refuses that ask and names `await`. The agent recovers, but only by way of a refusal.
   - Fix: name `await` in the mail when the standing is awaiting, as card 13's note does.

5. **LOW: cards 9, 12 and 13, no bound on re-asks when no parent resolves.**
   - With the parent orch down, every ask ends no-parent at once and the standing reads ask-again.
   - The session's ask, await and ask loop then spins without bound. Only hold notices, and the D5 class 5 count, would surface it.
   - None of the cards states a bound or a pause.

6. **LOW: cards 3 and 15, warnings logged on every call.**
   - `hubgeom.Autonomy` warns on every failed read. It runs on every Beam-Await wake (up to once a minute), every Slack hook turn, and from card 15's seam failures.
   - The logger guidance is to use an Episode for a failure that repeats each tick.

**Steps that progress, with nothing to report:**
- Card 1's `--set autonomy=` way forward.
- Card 6's `lym loom resume` for a decision halt (`shedadapters.DecisionHaltPrefix`, which `bouncerdecision.go:186` resumes the same way). The way forward itself works; finding 1 is about what the resume then does.
- Card 11's and Finalize's closed halts, with reopen, then goto PR-Review or resume.
- Card 12's refusals 1, 2 and 4, and refusal 3 assuming the record's `StrandName` matches the session's strand env (I did not check this).
- Card 13 cases 3 and 4.
- Card 14's reset error, which is idempotent on retry.
- Card 15's failed post, retried on the next wake.

**Tool calls:**
- Grep of `DecisionHaltPrefix` and `haltForParent`: to check whether resume admits a decision halt, which my context lacked.
- Read of `verify.go` 440-700 (`judge`, `resumeJudgeSession`): the resume path was not in my context.
- Read of `verify.go` 820-930 and `agreementGate`: to see how `runAgent` and the gate run on resume.
- Read of `MultiLLMProducer.Call`: to see whether resuming a judge that already ended starts a fresh spawn.
- Grep of the row's closed halt text and `resume.go`'s refusals: to check whether card 13's named steps can succeed.
