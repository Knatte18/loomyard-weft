# Bouncer CIRCLING escalates to the parent; loomcli smoke tier green

`loom-bouncer-rounds` · Task · bug, enhancement, loom, orch

# Goal

A Bouncer round loop does not halt a run for a human on something the parent orchestrator can settle, and the loomcli smoke tier is green on `main` before further loom runs depend on it.
`discussion-recurring-finding` and `circling-to-parent` are the same `board-model` halt seen from the anomaly report and from the Bouncer's verdict.

# Decisions from the task discussion (operator, 2026-10-05)

- **Checkpoint judging.** Below a checkpoint round the judge rules only `CONVERGED` or `CONTINUE`; it never rules `CIRCLING` there.
  At the checkpoint it decides whether more rounds are worth it: with progress it extends, never past `max_bounces`; with no progress (the same finding keys unchanged or reopened) it rules `CIRCLING`.
  A rising finding count alone is never circling.
- **Both numbers are configuration.** The checkpoint round (default 3) and `max_bounces` are configuration values;
  `max_bounces` lives in the recipe (`contracts/recipes/loom-recipe.yaml`) today, and Discussion settles where both live.
- **Escalation goes to the parent.** A `CIRCLING` ruling, and a budget spent without convergence, message the parent orchestrator the way a parent-review request does: a fork decides accept or continue and adds whatever design calls are needed.
  The operator is asked only when the parent cannot settle it, or the run has no parent.
- **The parent's decisions enter the decision record.** `post-discussion-additions` is part of this task: a verb records a parent's or operator's decision in the decision record, so plan review reads it as part of the answer key.
- **No GitHub issue for a recurring finding.** A recurring ledger finding goes to the parent as part of the circling escalation, not as a `selfreport` anomaly issue (#359 arrived with empty state and history).


## Discussion-Bouncer finding stays open across three rounds

From note `discussion-recurring-finding` (issues #359).

Imported from [issue #359](https://github.com/Knatte18/loomyard/issues/359).

### recurring-finding: a ledger finding stayed open across multiple rounds

**Slug:** board-model
**Parent:** main

**State:** 
**Current producer:** 

**Error:**
```

```

**History:**
no history entries

**Recurring finding:**
- Bouncer row: Discussion-Bouncer
- Ledger key: fold-target-semantics-open
- Rounds: [1 2 3]
- Status: open

## CIRCLING escalates to the parent orchestrator, later and on recurrence only

From note `circling-to-parent`.

Seen 2026-10-04 in `board-model`: Discussion-Bouncer judged round 2 CIRCLING because gating findings rose from 1 to 2, both in the D10 intake bound that round 1's fix had widened; the run halted for a human.
The hub settled it with five short design calls and the run went on; no operator input was needed.
Operator (2026-10-04): halting on that is disproportionate.
Idea: CIRCLING is judged no earlier than round 3 and only when the same finding keys recur unchanged, not on a rising count; and a CIRCLING verdict escalates to the parent orchestrator like a parent-review request (message, fork decides accept/continue with design calls), reaching the operator only when the parent cannot settle it.
Related: `post-discussion-additions` (a verb for the parent's decisions to enter the decision record, instead of the driver editing it by hand).

## loomcli smoke tier: five bootstrap/strand tests fail on main

From note `loomcli-smoke-failures`.

`go test -tags smoke ./internal/loomcli/` fails five tests on `main` (seen 2026-10-03 at `6a74ee1be`, during the PR-Gate review of #347, which fails the same five).
Each expects one `loom-status` or driver strand and finds none:

- `TestSmokeBootstrap_BringsUpSessionStrandAndDriver` — `smoke_test.go:530`: status strands named "loom-status" = 0; want exactly 1
- `TestSmokeBootstrap_SecondInvocationDoesNotSpawnASecondDriver` — `smoke_test.go:592`: status strands named "loom-status" after two bootstraps = 0; want exactly 1
- `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` — `smoke_driverstrand_test.go:216`: driver strands after the first bootstrap = 0; want exactly 1
- `TestSmokeStep_LeavesAPreexistingStatusStrandUntouched` — `smoke_drivingsurface_test.go:100`: status strand guids after step = []; want exactly the one added before the step
- `TestSmokeStart_LLMRunRemovesEveryStatusStrand`

Either the smoke tests are stale against the current bootstrap, or bootstrap no longer brings up the status and driver strands it should.
Nothing runs the smoke tier in the normal verify, so the failures went unnoticed.


From GitHub #348 (intake 2026-10-04).

## Scope added after Discussion amends the decision record

From note `post-discussion-additions`.

Seen 2026-10-03 in `agent-self-reflect`: the operator added two scope items after Discussion had been approved, and the hub relayed them to the Plan-Write agent.
On the hub's instruction, Plan-Write put them in the plan overview as "operator additions", and plan review then removed them, because the plan must follow the decision record and the record did not carry them.
PR-Gate rejected the PR for the missing items, and the driver had to amend the decision record by hand before PR-Rework.

Idea (operator): a scope addition after Discussion amends the decision record, not the plan.
The agent that receives it (Plan-Write, or whoever holds the run) extends the discussion with the new decisions before planning on them, even if that is a little messy.
Possible shape: a lyx verb that appends an "operator addition" decision to the record and re-runs the mechanical discussion check, so plan review reads the addition as part of the answer key.
