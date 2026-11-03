```yaml
lens: fix-proposal
outcome: reported
```

The F1 proposal is right in the code, but the test row added for it passes with the defect in place.

**Findings**

1. **MEDIUM — `waiting_test.go`, row "a waiting turn end over a shell already past the bound counts done and types nothing"**
   - **Claim:** the row cannot tell the fixed code from the defective code.
   - **Evidence:**
     - The row's gate passes. A gated Done with a passing gate reaches `handleGatedBoundary` (`wait.go:357`) and returns `finished` before `run.sendShellNudge()` at `wait.go:394`.
     - So the nudge queued by the defective order is never sent, and `SendTextCalls` stays at 1 (tick 1's failed send) with or without the fix.
     - The review's premise only shows when the gate fails with budget left: `handleGatedBoundary` returns unfinished after the re-prompt, and the loop falls through to `sendShellNudge`.
   - **Needed:** a gate that fails once, with the re-prompt sent and an assertion that no nudge text follows.
   - **Not run:** I cannot revert the fix read-only, so this is read from the code, not from a red run.

2. **LOW — `wait.go:934`, `outcome, heldTurnEnd, err := run.expiredTurnEnd()`**
   - **Claim:** the local `heldTurnEnd` shadows the type `heldTurnEnd`, which `wait.go:920` and `wait.go:948` use in the same function.
   - **Evidence:** the shadow compiles because the block never uses the type again. It reads badly, though, and breaks if someone adds `&heldTurnEnd{}` there.
   - **Alternative:** `held` is already taken in the enclosing scope at `wait.go:925`, so use `_` for the unused held result. `expiredTurnEnd`'s gated branch only returns `""` or Done.

**Proposals that hold**
- **F1, the `outcome == ""` guard:** sound. `expiredTurnEnd` on the gated path returns only `""` or Done. When it returns `""`, `waitingTasks` is intact, so `queueShellNudge` still sees the shells.
- **F2:** the wording is fine.

**Tool calls**
1. `grep -n "sendShellNudge()" -B14 -A4 wait.go`: I needed to see where the nudge is sent relative to the gated Done return, which my context lacked.
2. `Read wait.go 335–380`: I needed to confirm `handleGatedBoundary` returns `finished` before line 394 on a passing gate, which my context lacked.
