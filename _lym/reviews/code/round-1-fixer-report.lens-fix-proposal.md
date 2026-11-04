```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer applied as written (F1, F3, F4, F5, F6) and report findings.

Proposals checked: F1 `clearPendingSend`, F3 early return when covered, F4 `RemoveAll` on error, F5 revert, F6 row removal. F1, F4, F5 and F6 hold up.

- F1: `retryLanded` already cleared the record and returned false when no turn start carried the prompt text. Clearing it first changes nothing for the draft-in-box case (`ErrSentTextInDraft`). The operator-turn case now types the line, as intended.
- F4: `sessionDir` is minted per token, so `RemoveAll(absDir)` cannot delete a directory that existed before the open.
- F5, F6: no breakage.

**MEDIUM — F3: a covered attach can lose the deferred prompt for good**
- Changed text: `deliverPromptOnAttach` (`hold.go`, the added `|| run.covered`) and its new doc line "the run's next held turn end tries instead"; `wait.go`, the `run.observePause()` and `run.deliverPromptOnAttach()` calls before the loop.
- Summary: the early return drops the only attach-time delivery attempt, and nothing retries it once the pause lifts.
- Evidence:
  - `deliverPromptOnAttach` is called once, before the `for tick` loop, so nothing in the loop retries it.
  - The remaining delivery path is `deliverPromptAtHold`, reached only at a held turn end (`wait.go` lines 321-332).
  - The attach case exists for a run whose launcher died before the prompt went out, so the agent has had no turn and is idle.
  - An idle agent produces no turn end, so the prompt is never typed. The run waits for output files until its timeout, with no notice either.
  - Before the fix, `sendThroughPause` waited out the pause and typed the line. That worked, apart from the missing liveness polling the review raised.
- The review's alternative, retrying in the loop once `run.covered` clears, would fix both the stall and the lost delivery. The new test row ("a covering pause types nothing at Wait entry") pins the lost delivery as correct and never checks that a later delivery happens.

Tool calls made: none. My context already held every file the lens needed.
