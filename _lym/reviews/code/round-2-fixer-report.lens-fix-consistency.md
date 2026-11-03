```yaml
lens: fix-consistency
outcome: reported
```

Lines 868–869 are now stale (nudge queued only when nothing is done). Checking the test doc comment lines I saw earlier: "Every row finishes done through the gate at the plain turn end with no shell left over" is also stale for the new row.

**Task:** check the change set (F1 reorder in `wait.go`, new test row, F2 doc wording) for stale siblings or twins.

**Findings**

1. **LOW** — `internal/shuttleengine/wait.go:867-869` (the `pollEventsTick` doc, not edited by the fix).
   - It still says the tick "queues the shell nudge and returns what expiredTurnEnd answers". The code now queues the nudge only when `expiredTurnEnd` returns an empty outcome.
   - The old wording implies a nudge on a Done tick, the exact behavior F1 removed.
   - Suggested wording: "returns what expiredTurnEnd answers, queueing the shell nudge only while it answers nothing".

2. **LOW** — `internal/shuttleengine/waiting_test.go`, the doc comment of `TestWait_GatedShellNudge` (around lines 516-520).
   - It says "Every row finishes done through the gate at the plain turn end with no shell left over".
   - The new row "a waiting turn end over a shell already past the bound counts done and types nothing" finishes Done at a waiting turn end with the shell outstanding, and sets `wantDoneOverShell` to skip the `EndedShells` check.
   - The comment contradicts the row, and the opening sentence ("types one line", "types nothing once that line was sent") does not name the new case either.

3. **LOW** — `waiting_test.go` helper comment for `ageShellThen`.
   - It says "reads the clock from the run's fixture through shellAger, set per subtest". `shellAger` is a closure variable assigned in the loop, not a fixture field.
   - The shared mutable variable is fine because the test is non-parallel, but the comment's wording is loose.

**Checked and consistent**
- `websterengine/doc.go:641` now reads "With every output file present". Its other uses ("Master's output files") are unpinned.
- `shuttleengine/doc.go:441` and `spec.go:124`, which mention the gated nudge and expiry, still agree with the code.
- `queueShellNudge` and `sendShellNudge` docs (wait.go:1172-1198) make no claim about when the nudge is queued relative to expiry.

**Tool calls**
1. Bash grep of `nudge` and "With both present" across `internal` (non-test): to find twins of the changed text that my context lacked.
2. Read `wait.go:860-873`: to see the full `pollEventsTick` doc, since my context only held the span from line 880.
