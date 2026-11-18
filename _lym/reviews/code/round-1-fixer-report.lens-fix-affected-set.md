```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise findings' own searches (F1, F6, F9, F10) against the change set.

**Findings**

1. LOW, `hold.go` `retypeLostPrompt`, F6. The retype only declines to start once the deadline has passed.
   - Its send goes through `typePendingPrompt` at `run.go:1001`, which still uses `run.newSendContext()` with no deadline, so `awaitIdleSession` and `pasteVerified` are not cut to the run deadline.
   - A retype that begins just before the deadline can still run a full idle wait and submit window past it. That is the overrun the review described for `Wait`'s retype.
   - The review's `fix` text asks only for the early return, so this matches the proposal, but the finding's premise is not fully closed.
   - The same applies to the other `typePendingPrompt` callers, `deliverPromptAtHold` and `deliverPromptOnAttach`, which were outside the finding.

No other findings. The other sets are covered:
- **F1:** `promptSendOffset` is written at both sites that set `promptSentAt` (`run.go:883` and `run.go:1005`). Its only reader is `hold.go:205`. The remaining mentions of the prompt offset in comments and docs (`doc.go:174`, `doc.go:274`, `run.go:947`, `run.go:970`, `hold.go:120`) describe other readers of the persisted `PromptOffset`.
- **F9:** the only text naming `checkLivenessTick` as the capture's source was `wait.go:805`, and it is reworded.
- **F10:** the only "no later call retypes" text was the `promptDelivered` comment, and it is fixed.
- **F6 elsewhere:** the other `sendThroughPause` callers are the skill-load sends at `loadturn.go:140` and `loadturn.go:170`. They are not prompt sends and were outside the finding.

Findings I left unchecked: F2, F3, F4, F5, F7 and F8. They are about tests or one local latch, not a set of sites.

**Calls made**
1. `grep` for `promptSentAt`, `promptSendOffset`, `typePendingPrompt(`, `sendThroughPause(` and `sendStartPrompt(` in non-test code. My context lacked the full set of callers and setters.
2. `grep` for `checkLivenessTick recorded`, `no later call`, `past the prompt offset` and `lastStartupCapture` across `internal`, `docs` and the markdown files. I wanted any twin comment or doc sentence that the F1, F9 and F10 rewrites could have missed.
