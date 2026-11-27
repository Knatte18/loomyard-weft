```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals as the fix carried them out (fix-proposal lens).

**1. MEDIUM: `describereview.go` `check`, the `describe-review.sent` write (F12)**
The record is written before the findings reach the writer, so an undelivered re-prompt loses them.
- In `handleGatedBoundary` (`internal/shuttleengine/wait.go`), a re-prompt failing with `ErrSessionBusy`, `ErrSubmissionNotLanded` or `ErrPaused` spends no attempt. It sets `unsentReprompt` and clears `gateVerdict`.
- The next tick calls `evaluateGate`, so `check` runs again, finds `sent` and returns `Passed`. The run then finalizes with the findings never typed.
- The old in-memory `r.sent` already did this within one process. The persisted record extends it across a re-attach, which used to send the findings again. A driver that dies between the write and the send now never delivers them.
- No test covers a failed delivery.

**2. MEDIUM: `burler-step-fix.md`, `{{template "loom-fragment-tooling-names"}}` (F11)**
The rule now reaches burler fixers whose files are about the tooling's own records.
- `composeFixPrompt` renders `burler-step-fix` for every fix round, Plan-Burler-Fix and Discussion-Burler-Fix included.
- Those fixers edit `_lym/plan` and `_lym/discussion`, which must name the task, the decision record, gates, tiers and cards.
- The fragment says "A repository file you add or change names nothing of the tooling … not … the task, the Sketch or the decision record … a gate, a tier". Nothing in it exempts `_lym` files.
- The darn writer's `_lym/landing/summary.md` stop report is in the same position: it must name `lym loom decision add` and `lym loom resume`, though whether that counts as a "repository file" is unclear.

**3. MEDIUM: `describereview.go` `check`, `r.now().Sub(readyAt) < r.timeout` (F5)**
The review's time limit runs on wall-clock time from a persisted stamp, so pauses, machine sleep and halts all count against it.
- Example: the writer's run ends in a timeout while the review is pending, and hours later `lym loom resume` re-attaches the live session. That path does not reset `ready`.
- The first check then passes with "time ran out" while the reviewer may still be working.
- A `lym loom` pause longer than 15 minutes does the same.
- The shuttle run deadline, by contrast, counts only awake, unpaused time (`shuttleengine/doc.go`).

**4. MEDIUM: `hold.go` `missingOutputsLine` / `remindOfMissingOutputs` (F10)**
The line is typed right after any operator-started turn ends, including one where the operator told the agent to wait or is still mid-conversation.
- Example: the operator types "hold on, don't write anything yet" and the agent answers and stops. lym at once types "… carry on with your task and write them."
- "Once what the operator asked for is done" reads as already satisfied there, so the agent can proceed against the operator.
- Before the fix nothing was typed.
- The review also offered a visible notice instead, which does not have this effect.
- I did not check whether `Send` holds back while the operator is active in the pane. The code comment and the new test don't address it.

**5. LOW: `prreview.go` `judge`, `if staleText` (F13)**
`editWarned` can stay set with no edit being attempted.
- `syncText` returns early when `pullRequestText()` fails, without clearing it.
- So after one transient edit failure, a description that then stops parsing blocks both approval and `pr-continue` on every poll until it parses again.
- The log shows only the one "no pull request text could be built" warning.

**6. LOW: `config.go`, `pattern.MatchString("")` (F20)**
The premise is only partly closed: a pattern such as `\b|lym` does not match `""`, so it loads. Its leftmost match on a line is still empty, so the line is admitted.

Tool calls (2 of 6):
- `grep` and `sed` on `internal/shuttleengine/wait.go`: the retry path for an unsent re-prompt was not in context.
- `grep` on `internal/seatengine/engine.go` and `internal/burlerengine/prompt.go`: which prompt composer renders `burler-step-fix` was not in context. The seat-engine matches led to no finding.
