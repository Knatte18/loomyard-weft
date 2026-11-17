```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that every refusal, next step and criterion the fix added can be reached and leads to progress.

1. **MEDIUM.** Changed text: D6, "Within `settleUndecided` it sits after the soft-cap auto-accept … and before the escalated-record re-escalation, the soft-cap rule and the circling escalation."
   **Summary:** a replay of a round that already escalated with `stalemate` and is still pending can run another round without the parent.
   **Evidence:** `settleUndecided` (`internal/shedadapters/bouncer.go:1355-1376`) re-escalates an escalated round only when `windowOpen || verdict == verdictCircling`. Otherwise it calls `ensureFocus(round + 1)` and returns Stuck. D6 counts "starting after the latest round that escalated with cause `stalemate`". On that replay round N is itself that latest round, so its count is 0 and the stalemate check does not fire. Below the soft cap with a CONTINUE verdict, the round then falls through to the CONTINUE path. That breaks D6's "it runs no further round without the parent." The fix placed the check before the re-escalation without saying that a round already holding a `stalemate` escalation record escalates again with that cause. I did not check whether an earlier step in `settleUnconverged` catches a pending escalation before `settleUndecided` runs. The new D6 replay criterion makes this path one the plan must handle.

2. **MEDIUM.** Changed text: D3, "A `decision`-class finding the parent answered … takes `resolved` … and is not carried, so a settled question is never escalated again." Also the new D3 criterion "a `decision`-class finding the parent answered takes `resolved` and is not carried, and the Bouncer does not escalate it again."
   **Summary:** the "never escalated again" outcome rests on the reviewer choosing `resolved`, and no Go check enforces it.
   **Evidence:** the round-check entry's refusal list (D3) refuses nothing when the reviewer marks an answered decision finding `open` and carries it. `routeDecisions` (`internal/shedadapters/bouncerdecision.go:83-96`) then escalates with cause `decision` again. So the criterion cannot be asserted from Go, and the re-escalation loop the finding named stays open. A way forward is for the round-check entry to refuse `open` on a `decision`-class finding whose D2 block shows a recorded parent answer, plus a criterion row for that refusal.

Tool calls beyond context:
- Grepped `internal/shuttleengine` for `PassOnCap`. I needed to know whether a fix check passing on its cap reports `Passed`, since D1's new "the round is not archived" depends on it and my context lacked the gate code.
- Read `internal/shuttleengine/gate.go:74-80` and grepped for `Passed`, for the same question. The let-through state exists, so I raise no finding there.
