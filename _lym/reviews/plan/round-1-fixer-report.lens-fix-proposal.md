```yaml
lens: fix-proposal
outcome: reported
```

no findings

I challenged each proposal the fixer applied as written (F1–F6) and found no case where one is wrong or breaks something elsewhere:

- **F1, F2 (card 1 docs):** After D1, Finalize reroutes only through `keepFixerCommits`, for a spent or over-cap fixer. So the HandsBack list keeping only "a spent or over-cap fix", and the `Finalize.Call` merge sentence saying there is no reroute on a conflict or a large merge, are both true. The pawl "or halts where that edge reaches no review" clause is untouched and still true.
- **F3 (card 2, batten docs):** After card 2, `restepSkips` marks a loom or darn escalate halt as well as a landing halt. "A landing halt or a loom or darn escalate halt" is therefore the true set for both the `InnerRunDeps.HandBack` doc and the `binaryRestep` doc.
- **F4 (card 1, reroute-cap clause):** `TestLoomRerouteCapWayForward` builds its halt from an over-cap fixer reroute, not a parent-branch one (`wiring_test.go:1074-1076`). Dropping the parent-branch fragment leaves the test's setup valid, and its other three fragments still match the constant. A halt an older build wrote keeps its own text, so dropping the cause loses nothing.
- **F5 (card 2, pawl way forward):** Without the clause, a pawl settle that goes past the caps at Finalize still halts over-cap, and that halt carries its own way forward. The deleted clause promised a halt that does not come at a pawl Parent-Align whose Code-Review budget is above 0. Removing it leaves nothing false.
- **F6 (card 3, deleted row):** In `driverules.go`, "an awaiting review escalation" (line 224) and "a blocked stop at a row the arbiter judges first" (line 234) both choose `driveActionArbiter`. The driver then handles an arbiter error the same way whichever rule chose it (`drive.go:285`). So the existing arbiter-off row (`drive_test.go:301-304`) pins the error path for the judges-first route too, and card 3's Bound sentence on an arbiter that is off still has a test behind it.

Tool calls:
1. grep of `JudgesFirst` in `internal/sheddriver`, plus `wiring_test.go:1059-1110`: I needed to know how the reroute-cap test builds its halt (F4) and where the driver reads `JudgesFirst` (F6), and neither was in my context.
2. grep of `ArbiterFirst` and arbiter errors in `internal/sheddriver`: to find the rule that reads `ArbiterFirst` (F6), not in my context.
3. `driverules.go:215-250` and the `driveActionArbiter` sites in `drive.go`: to compare the action the two arbiter rules choose (F6), not in my context.
