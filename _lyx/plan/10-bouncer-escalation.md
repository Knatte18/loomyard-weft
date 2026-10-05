# Card 10 — The Bouncer escalates a CIRCLING ruling or a spent budget to the parent

**Edit:**
- `internal/shedadapters/bouncer.go`
- `internal/shedrecipe/entries_bouncer.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedadapters/doc.go`
- `docs/overview.md`
- `internal/shedadapters/bouncer_circling_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`

**Create:**
- `internal/shedadapters/bouncer_escalation_test.go`

**Uses:**
- `internal/shedadapters#EscalationCause`
- `internal/shedadapters#writeEscalation`
- `internal/shedadapters#readEscalation`
- `contracts/stencils/bouncer/bouncer-template-escalation.md`
- `contracts/stencils/bouncer/bouncer-template-parent-notice.md`
- `internal/shedengine/producer.go`

**Intent:**
D4, D5 and D7: a review segment's Bouncer never hands Shed the `Stuck` that would block on a spent budget; it escalates to the parent instead, and it escalates a guarded `CIRCLING` the same way.

`BouncerConfig` gains `DecisionRecordPath`, the absolute decision record path the brief names; the Bouncer entry passes `env.DecisionRecordPath` (documented on `Env` as also read by the Bouncer entry).
An empty value is legal and only degrades the brief render.

`settle`, for a `CONTINUE` or a guarded `CIRCLING` verdict on round N:
1. A decision file for round N is acted on first, whatever the budget:
   `accept` settles and returns `Done` after `Approve` and `Commit`, as the CIRCLING accept does today, for either cause;
   `continue` returns `Stuck` with the ledger pointer, marked `BudgetExempt` exactly when the decision's cause is `budget`.
2. Otherwise, the budget is reached when `Bounces` is wired and returns `ok` true, no error, and `count >= budget`.
   This is the same `>=` Shed applies to the history it read before appending, so the Nth judged round of a budget of N escalates exactly where Shed would block.
   No separate `review_max_bounces` comparison exists.
3. Reached: escalate with cause `budget`, also when the verdict is `CIRCLING`.
   Not reached and `CIRCLING`: escalate with cause `circling`.
   Not reached and `CONTINUE`: return `Stuck` as today.
   An unwired seam, an `ok` false or an errored read (warned) never escalates on budget, so that `CONTINUE` takes Shed's generic `bounce-budget-exhausted` block.

Escalating (one helper):
- When an escalation record for round N already exists, it rewrites nothing and returns the same `Awaiting` from the record's cause and notice.
- Otherwise it renders `bouncer-template-escalation` and `bouncer-template-parent-notice` through `stencilstore` and `stencil.Fill`, then calls `writeEscalation`.
  The notice interpolates only the slug and the brief path.
  A failed render warns and degrades to the plain Reason with no notice, while the record frontmatter is still written.
- It calls `ensureFocus(N + 1)` and returns `Awaiting` with the ledger pointer, `ParentNotice` set to the notice, and a Reason naming the segment, round and cause, the brief path, the `lyx loom circling` verbs with the slug, and the `lyx loom start` resume.
  The old clause that a continue will block on the budget is removed, and `circlingReason` becomes this cause-aware Reason.
- `settledGeneration` treats any round whose decision is a settled accept as settled, whatever its verdict, so a re-entry after a budget accept clears and re-seeds as a CIRCLING accept does.

What this lets through: a review segment past its budget proceeds only by a recorded decision.
The exemption covers only the Bouncer's `Stuck` for a `continue` on a budget escalation; the round producer's matching `Stuck` is Card 11's.
Each further round needs its own decision file, and `lyx loom goto` still grants a fresh budget.
Shed's generic block is unchanged and stays the backstop for every other row.

Docs in the same commit: `doc.go` replaces the CIRCLING/budget-sentence description with the escalation (both causes, the brief and notice files, `parent_notice`, decision-driven settle, the exemption), and `docs/overview.md`'s circling paragraph describes both causes, the parent notice, the parent's fork deciding and resuming with `lyx loom start`, and a batten-driven child being resumed the same way.

Tests in `bouncer_escalation_test.go` with a stub `Bounces` seam and seeded stencils:
- at count == budget a `CONTINUE` returns `Awaiting` with cause `budget`, a brief, a notice file and `ParentNotice`, not `Stuck`;
- a `CIRCLING` at a spent budget escalates with cause `budget`;
- below the budget a guarded `CIRCLING` escalates with cause `circling` and a `CONTINUE` returns `Stuck`;
- an unwired seam, `ok` false, and an erroring seam each return a plain `Stuck` at a spent budget;
- a recorded `continue` on a budget escalation returns a `BudgetExempt` `Stuck`, and on a circling escalation a counted one;
- a recorded `accept` on either cause returns `Done` after `Approve` and `Commit`;
- a re-call at an undecided escalation leaves the record and notice files byte-identical and returns the same `Awaiting`;
- a missing escalation stencil still returns `Awaiting` with the plain Reason and a written record.

`bouncer_circling_test.go` follows the new Reason and drops its budget-sentence cases.

**ImpactSummary:** Changes how every review-segment Bouncer ends a non-converged round at its budget and on CIRCLING; Shed and PR-Gate are untouched.

**Commit:** 10: The Bouncer escalates CIRCLING and a spent budget to the parent
