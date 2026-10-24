# Card 21 — darn-describe-entry-gate

**Edit:**
- `contracts/recipes/darn-recipe.yaml`
- `internal/loomrecipe#NewDarn`
- `internal/loomrecipe#DarnRouting`
- `internal/loomcli#rerouteReviewed`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestDarnRouting`
- `internal/loomrecipe#TestZeroBudget_BouncerSettlesAtOnce`
- `internal/loomrecipe/doc.go`

**Delete:**
- `internal/loomrecipe#applyDarnReviewBudget`

**Uses:**
- `internal/loomrecipe#stampZeroRounds`

**Intent:**
Darn's edge rewrite for a Code-Review budget of 0 goes (D10): the darn perch takes the same budget stamping as loom's, and Describe leaves itself out through its entry gate.

`NewDarn` stamps `zero_rounds` on the Code-Bouncer row through `stampZeroRounds` when the Code-Review budget is 0 and applies the segment's budget to the perch rows as loom's `New` does; `DarnRouting` applies the same budget with no edge rewrite; `applyDarnReviewBudget` goes.
Darn-Write's Done keeps routing to Code-Bouncer and Finalize's reroute to Code-Bouncer, so with a budget of 0 the Bouncer settles approved at once.
Darn's Describe row gets `entry_gate: budget.Code-Review`, so with a budget of 0 Describe is skipped and the writer's own change description stands, as today; with a budget above 0 the run is unchanged.
The recipe's header comment and Darn-Write's edge comment say so, and `rerouteReviewed`'s comment no longer names a rewrite.

`TestNewDarn_Routing` asserts that a budget of 0 keeps every edge, stamps `zero_rounds` on Code-Bouncer and builds Describe's entry gate answering false, and that a budget above 0 builds it answering true;
`TestDarnRouting`, over the routing projection, which builds no entry gate, asserts only that a budget of 0 keeps every edge and that the budget is applied to the perch rows; `TestZeroBudget_BouncerSettlesAtOnce` gains a darn row that builds `NewDarn` over its test env with a Code-Review budget of 0 and calls the built Code-Bouncer producer directly, as the loom row does, asserting it settles approved at once.
That Describe is then recorded `skipped`, leaving the writer's description, follows from its entry gate answering false, which `TestNewDarn_Routing` asserts, and from card 17's engine rows for a false gate.
The loomrecipe doc's line saying a budget of 0 routes `Darn-Write`'s Done and `Finalize`'s reroute straight to PR-Review is rewritten to darn's budget-0 behavior: the perch settles approved at once and `Describe` is skipped by its entry gate.

**ImpactSummary:** Removes darn's budget-0 edge rewrite in favor of loom's stamping and Describe's entry gate; only darn's routing at a budget of 0 changes.
