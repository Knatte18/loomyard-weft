---
round: 2
ledger:
  - key: mid-merge-prime-stale-config-inherited
    rounds: [1]
    status: resolved
  - key: preflight-stop-missing-from-decision-9-bound
    rounds: [1]
    status: resolved
  - key: decision-14-role-membership-contradiction
    rounds: [1]
    status: resolved
  - key: hub-lock-wait-unbounded-reconcile
    rounds: [1]
    status: resolved
  - key: relocation-decision-14-rationale-and-open-risks
    rounds: [1]
    status: resolved
  - key: pair-forked-during-walk-escapes-reconcile
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: decision-7-mid-merge-skip-triggers-after-write
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: cost-risk-understates-skipped-worktree-rewalk
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's five keys are resolved: round 2's review raises none of them again, and its focus verdicts mark the `Preflight` bound, Decision 14's role membership and the wait bound not a defect.
The mid-merge prime case is resolved as the round-1 finding; its remaining edges are the new keys below.

Round 2 opens three keys.
`pair-forked-during-walk-escapes-reconcile` (design, MEDIUM) is the gating one: a pair forked during another holder's walk is missed while the stamp still goes fresh.
`decision-7-mid-merge-skip-triggers-after-write` (consistency, MEDIUM) is that the skip triggers on a commit refusal that fires after reconcile has written.
`cost-risk-understates-skipped-worktree-rewalk` (consistency, LOW) is that a stale stamp from a skipped worktree makes every start verb and `Preflight` re-walk.
