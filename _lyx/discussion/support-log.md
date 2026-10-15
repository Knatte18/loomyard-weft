# Support log — refusal-route-config-empty

## Interview

Autonomous mode: no operator answers; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

- Board entry read; no other board entry touches begin-batch routing or `lyx config` editing (`run-verbs-shared`, `loom-model-key-names` and `role-language-skills` touch loom or config but not these paths).
- `ErrPlanDrifted` is the begin-batch dispatch sentinel; record-batch and recover-batch carry `plan_drifted` for fingerprint changes.
- The Webster master stencil writes `outcome: stuck` on `plan_drifted`; the Webster row has no `on_stuck`, so the run halts with `escalation-to-human`.
- delete-before-reference already runs at the plan gates through `callerCoverageFindings`; dispatch runs the scan-only `LaterDeleteReferences`.
- Shed's `on_reroute` crosses segments, spends no bounce budget and is capped at three; `on_stuck` cannot cross segments.
- The loom Webster row sets `AutoRebaseline`, so a pending-card plan edit is accepted on re-entry.
- `configengine.EditPath` scaffolds an absent sparse layer empty and keeps whatever the editor leaves; `editOne` then syncs and reports `edited and synced`.

### Turn 2 — batch 1: Part 1 label and routing (auto-picked)

- Q1 label: new `plan_conflict` class with cause `plan_defect`/`tree_moved` split on "has a batch begun". Recommended and picked.
- Q2 routing trigger: a Go-written conflict record, not the master's stuck text. Picked.
- Q3 reroute target: `Plan-Burler-Review` via `on_reroute`. Picked.
- Q4 third board bullet (check at Plan-Bouncer/Batchifier): already covered by the plan gates; no new check. Picked.

### Turn 3 — batch 2: Part 2 config (auto-picked)

- Q5 guard: stdin terminal test in `editOne`, refusing with `--print`/`--set` as the way forward. Picked.
- Q6 empty buffer: whitespace-only result aborts and restores. Picked.

### Turn 4 — lens review

All six forks returned.

- board-fit 1 (MEDIUM, sheddriver not named): fixed — D3 states the driver no longer sees a rerouted conflict; rejected alternative added.
- board-fit 2 (LOW, run-pane-timing merge neighbour): fixed — Open risks.
- board-fit 3 (LOW, weaver inherits class): rejected — weaver is unbuilt; nothing to change.
- board-fit 4 (LOW, run-verbs-shared/orch-split): rejected — reviewer asked for no action.
- board-fit 5 (LOW, Turn 1 omits sheddriver): fixed via board-fit 1.
- leanness 1 (MEDIUM, rejected alternative in D3): fixed — cut.
- leanness 2 (MEDIUM, Auto-mode assumptions restate decisions): fixed — "every class routes" moved into D3; restating bullets cut.
- leanness 3, 4 (LOW, contrasts in D2/D3 rationale): fixed — cut.
- leanness 5 (LOW, Out bullet deliberation): fixed — reduced to the requirement; reasoning stays in Rejected alternatives.
- leanness 6 (LOW, tolerant-records hedge): fixed — bullet cut.
- leanness 7 (LOW, no-denied-recovery phrased as fact): fixed — phrased as a constraint.
- leanness 8 (LOW, docs listed twice): fixed — Scope points at D5.
- goal-scope 1 / premises 1 (MEDIUM, rerouted review may not spawn; inherits episode): fixed — D3 states the round N+1 premise, Acceptance criteria pins the spawn, Open risks names inherited exclusions.
- goal-scope 2 / scenarios 2 (MEDIUM, false positive loops to the cap): fixed — D3 adds the last-routed findings check, so a repeated conflict halts after one plan review.
- goal-scope 3 (LOW, menu beyond board wording): fixed — Auto-mode assumptions.
- goal-scope 4 (LOW, whitespace reading): fixed — Auto-mode assumptions.
- goal-scope 5 (LOW, decline of earlier check rests on parity): rejected — the repeat check now catches gate/dispatch disagreement after one review, and a dispatch-identical check at Batchifier only adds a check for the checker's own bug.
- pattern-fit 1 (MEDIUM, tracked record with no committer): fixed — record moved to `.lyx/webster/`, never tracked.
- pattern-fit 2 (MEDIUM, write failure as new refusal variant): fixed — a write failure only logs; refusal unchanged; run halts as today.
- pattern-fit 3 (LOW, recipe names the path literally): fixed — Constraints require a test pinning the path.
- pattern-fit 4 (LOW, reroute-cap criterion re-tests shed): fixed — criterion replaced by recipe-edge validation.
- pattern-fit 5 (LOW, no halt note on reroute): fixed — D3 states it.
- premises 2 (LOW, gate is `ValidatePlan` with `done-card-edited`): fixed — D3 names it as a second bound.
- premises 3 (LOW, re-begun first batch counts as moved): fixed — D1.
- premises 4 (LOW, menu reads a choice before `editOne`): fixed — D6 guards before any choice is read.
- scenarios 1 (MEDIUM, stale record on auto-rebaseline refusal): fixed — Run removes the record first; reroute only on the stuck-outcome arm; acceptance row added.
- scenarios 3 (LOW, cancellation on reroute arm): fixed — D3 and acceptance row.
- scenarios 4 (LOW, failed write unpinned): fixed — acceptance row.
- scenarios 5 (LOW, hub-wide editor path): fixed — acceptance row.
- scenarios 6 (LOW, stale record forces edits after goto): fixed — Open risks states the stronger consequence (forced edits) for a record left by a halted Plan-Review followed by an operator goto; no removal hook on goto is added, since the operator who runs the goto sees the halt that left it.

## Rejected alternatives

- Keep `plan_drifted` and add a sub-field: the class name itself is the wrong cause, and the stencil and spec key on it.
- Per-finding cause attribution by diffing finding files against the run's start commit: not every finding carries a file, and the coarse split names both board cases correctly.
- Webster `on_stuck` to a plan row: `on_stuck` cannot cross segments, and it would catch every Webster stuck, fabric-sync failures included.
- Reroute to `Plan-Bouncer`: its judge does not see the conflict and may approve unchanged.
- A new dedicated plan-fix row: duplicates the Plan-Review ring and its `plan` gate.
- Let the Webster master edit the plan or run rebaseline itself: breaks its never-edit-the-plan role.
- Route on the master's quoted `stuck_reason`: agent text, not a Go fact.
- Store the conflict in `state.json`: the plan reviewer reads it as text; a markdown file serves both readers.
- Add the delete-before-reference check at Batchifier: it would check the same tree the plan gates already checked.
- Leave the stop to the Go driver's tackler (`internal/sheddriver`): a cold tackler would rediscover what the refusal already names, and the plan edit belongs to the Plan-Review ring with its gate.
- Do not reroute `plan_defect` at all: the repeat check already halts a gate/dispatch disagreement after one review, and a real authoring defect the gate missed is still a plan edit.
- Write the last-routed file at `Plan-Burler-Review`'s entry, so only a conflict a plan review saw counts as routed: `BurlerReviewProducer` serves every review segment and would need a plan-only hook, and a capped conflict's way forward is the same hand edit either way, so the adapter keeps the write and D3 states the capped case.
- Config: print the file instead of refusing when there is no terminal: a refusal naming `--print` makes the caller's intent explicit and avoids a silent mode switch.
- Config: treat an unchanged file as abort too: not asked for; an unchanged non-empty file writes nothing harmful.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): begin-batch's class is `plan_conflict` with cause `plan_defect` or `tree_moved`. Rationale: the fingerprint has already matched, so the plan did not drift; the begun-batch test is cheap and splits the two board cases.
- Q2 (resolved, auto-pick): routing rests on a conflict record begin-batch writes, removed at Run entry once Run holds run.lock and as every begin-batch's first step. Rationale: Go-written, bounded to one Webster call.
- Q3 (resolved, auto-pick): Webster `on_reroute: Plan-Burler-Review`, plan reviewer instructions name the record. Rationale: existing ring with a mechanical gate; reroute cap bounds loops.
- Q4 (resolved, auto-pick): no new delete-before-reference check at Plan-Bouncer or Batchifier. Rationale: plan gates already run it.
- Q5 (resolved, auto-pick): stdin-only terminal guard on every `lyx config` editor path. Rationale: agents' shells have no terminal on stdin.
- Q6 (resolved, auto-pick): empty or whitespace-only editor result aborts; comment-only saves. Rationale: matches the board's "never saved over a layer".
- Q7 (resolved, auto-pick): every blocking dispatch finding class routes, not only delete-before-reference. Rationale: they share one way forward.
