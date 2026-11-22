```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every way forward the fixer's change set introduced can be carried out from the state it names and leads to progress.

**MEDIUM: the takeover's first step does not leave the run where goto admits it**
- Changed text: D4, "`lym tender pause <slug>` stops the waiting run, which leaves it paused at PR-Review, where goto admits the row, then `lym loom goto` sends it back to PR-Review."
- Summary: `lym tender pause` writes a pause-record entry. PR-Review's `holdWhileCovered` then holds inside the row's call: it stops the session and sleeps while the pause covers the run, and never returns a paused outcome. The run's status stays running, and `gotoAdmitted` admits the reference row only on a paused, blocked or failed run, so the named goto is refused.
- Evidence:
  - `internal/landingshed/prreview.go` `holdWhileCovered`: the loop sleeps while covered and returns only once uncovered.
  - `internal/tenderengine/pause.go`: it only records an entry (`pauserecord.Entry`) and interrupts sessions.
  - `internal/shedengine/goto.go` `gotoAdmitted`: "paused, blocked and failed admit the reference row itself".
  - The `lym loom pause` Long help in `internal/loomcli/cli.go`: `--after` ends a mid-row wait only for a row that waits for the parent lock, and PR-Review takes no parent lock (its row doc says so).

**MEDIUM: a timed-out PR-Review ask's refusal names a way forward that cannot end the review**
- Changed text: D4, "The answer verb is unchanged… An ask that reaches its deadline unanswered is set timeout… the answer verb then refuses it, and the session asks again with `lym loom parent-review ask`."
- Summary: because the verb is unchanged, its past-deadline and ended refusals give the parent `parentReviewWayDecision` as the way forward ("a call the parent still wants made goes in with lym loom decision add <slug> --by parent"). A decision entry never ends PR-Review, which ends only on a satisfied PR-Review ask. The refusal sends the parent orch somewhere that makes no progress instead of telling it to wait for the session's re-ask.
- Evidence: `internal/loomcli/parentreview.go` `parentAnswerRefusalText`, the `ParentAnswerPastDeadline` and default cases, and `parentReviewWayDecision`.

**MEDIUM: the review left open by a cleared continue flag has no cue that leads to a new ask**
- Changed text: D4's new `judge` bullet, "Otherwise the row clears the continue flag and the review stays open, so a commit made after a passing `pr-continue` is gated and waits for a new ask at the new head."
- Summary: the bullet names nothing that tells the session's agent the flag was cleared or that it must ask again. After a passing `pr-continue` the agent is done. The newest PR-Review ask is done and satisfied, so it is not "unanswered at the current head", and the hold notices D4 relies on raise nothing that the parent can answer: no ask is open.
- Evidence:
  - The D4 Bound sentence that hold notices re-raise only an unanswered ask.
  - `pr-continue`'s "otherwise they name asking the parent again" path, which runs only if the agent calls `pr-continue` again; nothing prompts that call.
  - `gateHead` in `internal/landingshed/prreview.go`, which mails the session only on a gate failure, never on a pass.

The other new ways forward check out:
- The closed pull request's refusal leads to the existing closed-without-merge halt (`judge`, StateClosed, no Continue).
- D3's not-sent and no-parent halts resume into a re-ask, and a persistent cause escalates through class 5.
- A re-ask after a timeout is admitted, since a timed-out ask no longer stands unanswered.

Calls made:
- grep for the tender pause verb: my context did not show what `lym tender pause` does to a run's status.
- Read the opening of `internal/tenderengine/pause.go`: needed to confirm the pause only records an entry and interrupts sessions, and sets no shed state.
