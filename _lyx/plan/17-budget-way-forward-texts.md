# Card 17 — budget-way-forward-texts

**Edit:**
- `internal/shedbuild#ShedPaths`
- `internal/shedbuild#NewShedFrom`
- `internal/shedbuild#TestNewShed_FieldCompleteAssembly`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#loomVerbTexts`
- `internal/shedcli#shedVerbTexts`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#Command`
- `internal/battencli#battenCLI.wire`
- `internal/loomengine#TestClassifyHalt`
- `internal/loomengine/anomaly.go`
- `contracts/stencils/shed/shed-template-driver.md`
- `contracts/stencils/orch/orch-template-role.md`

**Create:**
- `plan:internal/loomcli#loomBudgetWayForward` -> `const loomBudgetWayForward = "x"`

**Intent:** D3's told way forward and every text that names a goto as the way to a fresh budget, rewritten to D1–D3.

`ShedPaths` gains `BudgetWayForward`, copied onto `Shed.BudgetWayForward` by `NewShedFrom`.
`loomBudgetWayForward` is `run "lyx loom resume" in the task worktree: the resume grants the row one more budget; a goto keeps the spent budget and grants none`, told by both `loomCLI.wire` and `loomCLI.wireLightweight`, so a darn run, which shares loom's wiring, names it too.
Batten's wire tells `run "lyx batten run <slug>": the resume grants the row one more budget`.

The goto help texts (`loomVerbTexts`, `shedVerbTexts`, `battenVerbTexts` and batten's `Command` overview) drop "with a fresh segment budget" and "resets that row's segment bounce budget": a goto keeps the segment's settled verdicts and spent budget, resets only what the target row needs to run (the error, the transient mark, a pending resume label and, for goto alone, the pause conditions), records itself as the run's pause reason, and is refused onto a review or fix row of a settled review segment where the recipe tells that guard.
The driver stencil's "Two stops name `lyx shed goto`" list keeps only the missing-producer stop and names the told resume clause for an exhausted budget; the orch role stencil's "A run that needs a fresh budget at a producer" bullet becomes a resume of the budget halt, with goto kept for moving a run back; `anomaly.go`'s comment on the budget prefix no longer says the reason carries a goto way forward, and `TestClassifyHalt`'s budget row text names the resume clause.

`TestNewShed_FieldCompleteAssembly` asserts the told clause reaches the Shed, and card 2's told-`BudgetWayForward` row pins that a told clause becomes the halt's reason; the clause texts themselves are static prose, so no test pins them.

**ImpactSummary:** Wiring of one told clause in loom and batten plus help and stencil prose; no behaviour beyond the budget halt's reason text.

**Commit:** 17: texts: a budget halt names the resume verb, and no text calls a goto a fresh budget
