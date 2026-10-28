# Fixer report — discussion round 1

## Changes

- F1 (BLOCKING), fixed as proposed: D4 now hands `JudgesFirst` the stop's Reason, or for a stop built from a status its Error, as the budget-exhausted row reads both.
  The driver acceptance criterion now also covers a driver that starts or re-checks at a status-built escalate stop.
- F2 (MEDIUM), fixed as proposed with option (a): a pawl escalate stays a hand-back.
  D3 adds a `Deps.EscalateJudgedFirst` field, which `landingDeps` sets true on loom and darn and false on pawl; when it is false, the escalate halts with `LandingStopPrefix`.
  D4's pawl sentence is removed.
  The pawl limit now also appears in the Goal, the Scope bullet, D3's Bound, D5 (orch stencil and fixer report spec), the pawl Constraint, the Notes' drift-guard line and a new acceptance criterion.
- F3 (LOW), fixed as proposed: D4's Bound now says an escalate gives its fixer attempt back, so the fixer caps never stop the loop.
  It also says the loop spends the run's one arbiter budget, and every later arbiter-eligible stop then hands back to the orch.
- F4 (LOW), fixed as proposed: D3's prefix must now differ from both `LandingStopPrefix` and `shedadapters.DecisionHaltPrefix`.
- The support log gained a Turn 4 recording these fixes.

## Self-check

- fix-consistency, MEDIUM: the old escalate-prefix criterion was not limited to loom and darn, so it contradicted the new pawl criterion. Fixed as proposed: it now opens "On a loom or darn run".
- fix-consistency, LOW: D3's opening sentence was not limited to loom and darn. Fixed as proposed: it now opens "On a loom or darn run".
- fix-consistency, LOW: the support log's Turn 3 still recorded the old pawl claim. Fixed as proposed: that entry is now marked as superseded in Turn 4.
- fix-way-forward, MEDIUM: the pawl escalate halt kept today's way forward, `lym loom decision add`, which refuses a run with no decision record. Fixed as proposed: D3 now gives the pawl halt the way forward "settle the conflict on the task branch, then `lym loom resume`", and the pawl criterion says the way forward names no `lym loom decision add`.
- fix-claims: no findings.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

None.
