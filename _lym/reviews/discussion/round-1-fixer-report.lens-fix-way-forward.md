```yaml
lens: fix-way-forward
outcome: reported
```

I checked the ways forward this fix adds. Four of them cannot be carried out as the new text states.

1. MEDIUM — D4's `pr-continue` refusal leads nowhere when an ask is already waiting.
   - Changed text: D4, "With neither, it refuses, naming the way forward: ask the parent again", and D4's Bound, "`lym loom parent-review ask` is … refused while the newest ask stands unanswered at the current head".
   - Problem: when the newest ask was sent at the current head and has no answer yet, `pr-continue` tells the agent to ask again, and `parent-review ask` refuses that.
   - Fix: in that state the refusal should name waiting for the parent's answer, since the hold notices raise the ask again.

2. MEDIUM — D9 names a read that does not return the resolved value.
   - Changed text: D9, "its own worktree's value from `lym config beam --print`, run when it takes a D7 step".
   - Problem: `internal/configcli/doc.go:20-31` says a verb with no `--layer` addresses the module's own file, the worktree file, and that `--print --layer <name>` prints a layer's file verbatim. It is `lym config <module> --resolved` that prints the merged result.
   - Effect: `--print` misses a hub-wide `--layer default` true, so the orch reads the wrong value.
   - Fix: name `lym config beam --resolved`.

3. MEDIUM — D3's way forward after an expired judge ask is not shown to ask the parent again.
   - Changed text: D3's Bound, "the halt names `lym loom resume`, which asks the parent again".
   - Problem: the ask record belongs to the judge's spawn. `internal/driftengine/judgeparent.go:95-99` says "one at or past its deadline is set timeout and passes". Its deadline is also capped by `judgeAskLimit`, which is the judge run's start plus `judgeTimeout` less a margin (`judgeparent.go:61-68`).
   - Effect: a resume that keeps the same spawn finds the ask already marked timeout and lets the rulings stand. That is the bypass D3's Bound is meant to close, and nothing in the record says a resume opens a new ask.
   - Fix: D3 should state that the halt clears the timed-out ask, or that the resume starts a new judge spawn whose ask is new.

4. MEDIUM — the gate-failure mail still sends the session to the operator.
   - Changed text: D4, "With a mailed head standing and no such answer, it clears the mailed head, so the row gates that head again".
   - Problem: the session learns of this path from `gateRetryHint` (`internal/landingshed/prreview.go:716`). Its text says "and the operator has asked you to continue, run lym loom pr-continue".
   - Effect: under autonomy no operator sits in the session (`Interactive` is false), so the agent cannot take that step as written. D4 and D9 do not list the hint among the texts that take the autonomy exception.
   - Fix: list it among those texts.

Tool calls beyond my context:
- `grep`/`sed` on `internal/driftengine` (`verify.go` and `judgeparent.go`) and `internal/configcli/doc.go` lines 1-40, to check what happens to the judge ask on a resume and what `--print` prints; neither was in my context.
- `sed` on `internal/configcli/doc.go` lines 20-32 and `internal/driftengine/verify.go` lines 830-880, to read the `--print`, `--layer` and `--resolved` wording and how the gate builds the ask's deadline.
- `grep` on `judgeAskLimit` and the deadline and resume lines in `internal/driftengine/judgeparent.go`, to see whether a timed-out ask passes the gate and how its deadline is capped.
