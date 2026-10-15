# Card 7 — Plan-Write reads the relevant list first

**Edit:**
- `internal/loomengine#PlanTable`
- `internal/loomengine#TestPlanTable`
- `contracts/stencils/loom/loom-template-plan.md`

**Uses:**
- `plan:internal/loomengine#DiscussionRelevant`

**Intent:**
Per D4, Plan-Write reads the decision record, then `relevant.md` when it exists, then the code the list names that the plan needs, then explores further as needed.

`PlanTable` adds a table value `relevant_path: DiscussionRelevant(layout)` beside `decision_record_path`.
`plan.go`'s file header drops "Its sole input is `_lyx/discussion/decision-record.md`": the decision record is the producer's design input, `relevant.md` its code head start when present, and it reads nothing else from `_lyx/discussion/`, never the support log and never the board.

`loom-template-plan.md`'s Step 1 reads `{{.decision_record_path}}` as the only source of design and scope, then `{{.relevant_path}}` if it exists, as a head start that never adds scope and whose format the discussion spec defines;
a missing list is not an error, and the writer explores as Step 2 says.
It still never reads the support log or the board, and the "STOP if the decision record is missing or empty" rule is unchanged.
Step 2 says to start from the code the list names before exploring further.
The rework and prior-plan stencils are untouched.

`TestPlanTable` asserts `table.Values["relevant_path"]` equals `DiscussionRelevant(layout)`.

**ImpactSummary:** one new table value in `PlanTable` and comment and stencil wording; Plan's seats, outputs and gate are unchanged.

**Commit:** 7: Plan-Write reads relevant.md first
