# Bouncer CIRCLING escalates to the parent; loomcli smoke tier green

`loom-bouncer-rounds` · Task · bug, enhancement, loom, orch

# Goal

A Bouncer round loop does not halt a run for a human on something the parent orchestrator can settle, and the loomcli smoke tier is green on `main` before further loom runs depend on it.
`discussion-recurring-finding` and `circling-to-parent` are the same `board-model` halt seen from the anomaly report and from the Bouncer's verdict.


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
