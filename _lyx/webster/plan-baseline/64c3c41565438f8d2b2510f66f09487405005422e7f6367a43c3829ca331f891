# Card 11 — fabric-parent-holds-task-work

**Create:**
- `internal/fabricengine#ParentHoldsTaskWork`

**Uses:**
- `internal/fabricengine#parentHoldsTaskWork`

**Intent:**
The read behind the Finalize carve-out of InnerRun's and Finalize's owner checks (D6, D7).
`ParentHoldsTaskWork` exports `parentHoldsTaskWork` unchanged, and `finalizeEntered` keeps calling the unexported one.

**ImpactSummary:** fabricengine exports one existing read; no existing caller changes.

**Commit:** 11: fabric: export whether the parent already holds a task's work
