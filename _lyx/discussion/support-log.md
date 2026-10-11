# Support log — fixer-critical-of-findings

## Interview

The session ran autonomously, so there was no operator interview; each turn below is an exploration step and the pick it led to.

1. Board entry read: the brief asks that the fixer may dispute a finding whose proposed change would be wrong or make the code worse, and choose its own fix when the premise holds but the concrete change is wrong, with the judge ruling on disputes as today.
   Related entries: `burler-multi-llm` (rewires the round, keeps the review and Bouncer contract) and `plan-gaps` (a sanctioned card insert for the fixer); neither touches the dispute rule.
2. Read `burler-step-3-fix.md`, `burler-template-fix-orchestrator.md` and `burler-step-2-review.md`.
   Each finding's prose carries `**Issue:**` (the premise) and `**Fix:**` (the proposed change), so the two checks the brief wants have distinct targets.
3. Traced who rules on a dispute today: the judge stencil never reads the fixer-report; the next round's reviewer reads prior fixer-reports after its own review and "re-evaluates their deferred and disputed items", and the judge rules on that review.
   So "the judge rules as today" means this path, kept unchanged (D5).
4. Checked the convergence and carry-over paths (`decisionRuleMarker`, `carryover.go`): a dispute is no convergence signal, and a MEDIUM or BLOCKING closing-round dispute reaches `## Open risks` with the fixer-report path (`carriesOver` drops LOW and NIT).
   The converged closing text says each finding was "addressed by the round's fixer" and that no fresh reviewer "has seen the fix", which an every-fix-worse dispute falsifies; review round 2 raised this, and the text is reworded (D7).
5. Split the brief into two cases: a premise that holds but whose every remedy is harmful (a dispute), and a premise that holds with a wrong prescription (a fix with the fixer's own change). Picked the ordered three-way disposition (D1–D3).
6. Weighed whether the reviewer side is in scope. A broader dispute ground with a reviewer free to re-raise without engaging the evidence leaves the next fixer nothing to weigh, so the dispute is decided by repetition; added a one-sentence rule to `priorRoundsBlock` (D4).
   The judge rules by class and severity alone and does not read the answer.
7. Listed docs that state the old rule: `PATTERN-review-round` background, `burlerengine/doc.go`, `docs/overview.md`; the `PATTERN.md` one-liner still holds (D6).
8. Corrected a draft rationale that cited stale model defaults from commit 572ab84e8; `internal/loomengine/template.yaml` now sets both `review` and `fix` to `opus[medium]`.

## Rejected alternatives

- **Allow a dispute whenever the proposed change is wrong.** Rejected: a wrong prescription on a real defect leaves the defect; the finding must still be fixed, so this case is a fix with the fixer's own change, not a dispute.
- **A dispute ground of "the change makes the code worse" judged on the proposed change alone.** Rejected: it lets a fixer skip a real defect by attacking one bad remedy; the bound requires evidence against every addressing change.
- **A new `Fixed differently` fixer-report section.** Rejected: the fixer-report has no parser, the next reviewer judges the current target fresh, and an inline mark in the changes list carries the same information without a new format surface.
- **Have the judge read the fixer-report's `Disputed` section and rule on each dispute.** Rejected: the brief keeps the judge as today, the judge is a sit-rep agent that never opens artifacts, and the next fresh reviewer already settles disputes.
- **Accept the converged carry-over text as an open risk instead of rewording it.** Rejected: the text tells the plan writer a fix exists where an every-fix-worse dispute made none, and rewording one string is cheaper than a known false statement in the plan writer's sole input (D7).
- **Leave the reviewer untouched (fixer-only change).** Rejected: without D4 a re-raised dispute carries no answer the next fixer can weigh, so a persisting dispute is decided by repetition rather than on its merits.
- **Allow disagreement with the rubric or PATTERN as a dispute ground.** Rejected: the rubric and PATTERN are the fasit's binding rules, and part B of the review-fix pattern directive already forbids a fix that breaches a constraint.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Is "proposed change is wrong" a dispute or a fix? — Fix with the fixer's own change; only a finding no change can improve is disputed. Recorded as D1/D3.
- Q2 (resolved, auto-pick): What bounds the new dispute ground? — Cited evidence that every addressing change makes the target worse; severity, size, cost, taste and rubric or PATTERN disagreement stay forbidden. Recorded as D2.
- Q3 (resolved, auto-pick): Does the judge change? — No; the next fresh reviewer settles a dispute and the judge rules on its review, as today. Recorded as D5.
- Q4 (resolved, auto-pick): Is the next reviewer's prior-rounds instruction in scope? — Yes, one sentence: a re-raised disputed finding answers the dispute's evidence. Recorded as D4.
- Q5 (resolved, auto-pick): New fixer-report section for fixed-differently findings? — No; an inline mark in the changes list. Recorded as D3.
- Q6 (resolved, auto-pick): Does `PATTERN.md` change? — The one-liner stays; the `PATTERN-review-round` background changes. Recorded as D6.
- Q7 (resolved, auto-pick): Should the freedoms depend on the fixer's model? — No; model choice is `loom.yaml`'s and out of scope. Recorded under Auto-mode assumptions.
- Q8 (resolved, review round 2): Does the converged carry-over text change? — Yes; it says fixed or disputed and points at the fixer report, since an every-fix-worse dispute makes no fix. Recorded as D7.
