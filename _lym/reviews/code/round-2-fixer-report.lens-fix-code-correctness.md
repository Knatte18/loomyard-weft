```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code correctness of the `wait.go` and `waiting_test.go` changes. No findings are BLOCKING or MEDIUM.

**Walkthrough of the new row.** Input: `events: "WAIT\n"`, `sendErr` set, `steps: [ageShellThen(appendLine("WAIT"))]`.
- **Tick 1:** `pollEventsTick` reads WAIT, so `recordWaiting` stamps `shellFirstSeen` at the current time. In the gated shell-only branch with the output file present, `expiredTurnEnd` returns `""` because the shell is not past the bound. The new code then calls `queueShellNudge`, and the send fails on `sendErr`, so the shell stays out of `nudgedShells`. That is 1 send.
- **Sleep step:** `fc.asleep(ShellWaitBound(gateConfig))` advances the clock by `defaultBackgroundShellWaitMin` (10 minutes), well inside the 1-hour deadline. A second WAIT is then appended.
- **Tick 2:** the new waiting turn end calls `expiredTurnEnd`, which finds the shell past the bound and returns `OutcomeDone`. The new code skips `queueShellNudge`, so there is no second send and `wantSent: 1` holds.
- **With the defect in place:** the old order queued the nudge first. Because the failed send left the shell unnamed, `queueShellNudge` would queue it again, `sendShellNudge` would send a second time, and `SendTextCalls` would be 2. The row fails without the fix.

**Callers.** `pollEventsTick` keeps its signature, and `expiredTurnEnd` is unchanged. The other call sites (`len(data)==0`, `len(events)==0`, and the non-shell fallthrough) do not nudge, as before.

**Failure paths.**
- `expiredTurnEnd` never returns a non-nil error, so the `err` pass-through is inert.
- The held-turn-end return is `nil` on every gated path, so nothing is dropped.

**Findings**
1. LOW: `wait.go:934`, `outcome, heldTurnEnd, err := run.expiredTurnEnd()`. The local `heldTurnEnd` shadows the type `heldTurnEnd`, which this function uses at line 920 and `held := run.waitingTurnEnd` at line 925. It compiles, because the type is not used again in that block. It is a readability trap, and naming it `held` would match the sibling variable. The inner `err` also shadows the outer `err`.
2. LOW: `waiting_test.go`, the doc comment above `TestWait_GatedShellNudge`. It was not updated for the new row, so it does not mention that a waiting turn end already past the bound counts Done and types nothing. It also still says "Every row finishes done through the gate at the plain turn end with no shell left over", which the new row, with `wantDoneOverShell`, contradicts.

Tool calls: none. My context already held every changed file and the review.
