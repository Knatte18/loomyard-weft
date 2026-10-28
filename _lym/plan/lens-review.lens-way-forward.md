```yaml
lens: way-forward
outcome: reported
```

Way-forward lens on the plan: for each refusal, stop or recovery step the cards add, I checked whether its next step can succeed from the state it leaves.

**Findings**

- LOW. Card 2 (D3), pawl escalate way forward. The new text says "settle the conflict on the task branch, then `lym loom resume`". When the operator settles it by hand, that resume can halt a second time before it lands.
  Evidence: the escalate end writes no `Halt`, and `Finalize.gateAndFix` (`internal/landingshed#Finalize.gateAndFix`) only resets `Baseline` after an over-cap halt. So on the resume, a passing gate goes to `Finalize.judgeFixerChange`. That function measures `LineChange(rec.Baseline, head)`, and the range includes the operator's own commits. On a pawl run `RerouteReviewed` is nil (`internal/loomcli#landingDeps`). An operator settle that touches a test file or goes past `finalize_fix_max_lines` therefore halts over-cap with "review and fix on the task branch". Only a second `lym loom resume` lands it.
  The run still moves forward, but the stated next step is not the last one. Card 2 should name this in the pawl way forward, or say the over-cap halt may follow and its resume accepts the commits.

Per refusal, next step, progresses or not:
- Loom or darn escalate halt (Card 2): `lym loom decision add` then `lym loom resume`. Progresses. The arbiter may record with `--by arbiter` and the orch with `--by parent` (`internal/loomcli/decision.go:550-570`). The arbiter's `resume` needs a goto (`contracts/stencils/loom/loom-template-arbiter.md:79`), and goto to the halted row passes `settledSegmentGuard`, since Parent-Align and Finalize sit in no review segment. The Finalize goto guard refuses only a dirty parent, which has its own way forward. The re-run fixer has its escalate turn back.
- Arbiter hand-on, arbiter off or spawns spent (Card 3): the stop goes back through the existing hand-on path, and the halt reason's own way forward still applies. Progresses.
- LLM driver at a `restep_hold` stop (Card 4): the stop goes to the arbiter first (`contracts/stencils/shed/shed-template-driver.md:60`), then to the parent on a hand-on. Progresses.
- Batten re-step skip (Card 2): leaves the stop to the arbiter and the orch. Progresses.
- Card 1 conflict session stuck under the lock: a non-halting Stuck with the resolver's reason. The next Finalize's pre-merge step aborts the parked merge-in and stops the session (`Deps.AbortMerge`, `Deps.StopConflictSession`). Progresses.
- Spent or over-cap halt and reroute: unchanged.

Tool calls:
1. Grepped `shed-template-driver.md` and `loom-template-arbiter.md` for blocked-stop routing and resume/goto rules. My context lacked the stencil text past the `hand_back` lines.
2. Grepped `decision.go` for the `decision add` entry points. My context lacked who may run the verb mid-run.
3. Read `decision.go:75-100` and its refusal lines, for the same reason.
4. Read `settledGuard` in `settledguard.go`, to check that the arbiter's goto to the halted row is admitted.
5. Read `finalizeGuard` in `finalizeguard.go`, for the same reason.
