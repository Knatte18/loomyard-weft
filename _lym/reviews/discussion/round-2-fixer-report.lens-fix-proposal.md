```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer applied unchanged (lens: fix-proposal).

Proposals challenged: F1 (LLM driver skip, D5 stencil and guide entries, criterion), F2 (driver criterion rewrite), F3 (D3 scoped to batten), F4 (status spec entry in D5), F5 (Turn 4 move).

1. **LOW: the F2 criterion makes any fresh driver start re-step, but D3 and D4 only allow the build hand-over to.**
   - **Changed text:** `decision-record.md`, Acceptance criteria, driver bullet: "a driver starting at a halted escalate stop with no open attempt re-steps the row, and only a re-escalation reaches the arbiter".
   - **Problem:** this re-step happens on every fresh driver start, not just the build hand-over, so its scope is wider than the bound that D3 and D4 state.
   - **Concrete case:** a driver dies or is revived after the escalate halt is written but before the arbiter attempt is opened. The next driver goes through `checkBlockedStart`, then `Step`. That re-runs Finalize and a fresh fixer over the undecided collision, and that fixer may settle the collision itself and land with no arbiter or orch turn.
   - **Conflict with siblings:** D3, as F3 changed it, says "the driver's own build hand-over re-steps it once, per D4's Bound". D4's Bound names only the build hand-over ("while the driver lives, a new build re-steps the escalate stop once"). Neither covers a restart for any other reason, and neither covers repeated restarts.
   - **Evidence:** `internal/sheddriver/drive.go:181-186`: `resumeOpenAttempt`, then `checkBlockedStart`, then step. `internal/shedengine/run.go:221-223`: a blocked status does not short-circuit.
   - **Remedy:** widen D4's Bound, and D3's sentence, from "build hand-over" to "a driver start with no open attempt, the build hand-over included".

2. **No finding on F1.**
   - The LLM driver never forks at a `blocked` stop (stencil line 59–60), so the escalate skip has no fork re-launch case to collide with.
   - Arbiter eligibility still comes from `arbiterstop.Check`, which D3 already makes admit the escalate.

No findings on F3, F4 or F5.

Tool calls:
- Grepped `contracts/stencils/shed/shed-template-driver.md` for `blocked` and `fork` lines. My context held only that stencil's lines 90–150, and I needed its blocked-stop and fork rules to test F1's proposal.
