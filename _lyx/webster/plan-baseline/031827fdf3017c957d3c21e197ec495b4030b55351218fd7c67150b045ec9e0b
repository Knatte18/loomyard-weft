# Card 14 — attack-surface-question

**Prosa:**
- `contracts/stencils/loom/loom-rubric-discussion-review.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-template-discussion.md`
- `contracts/stencils/loom/loom-template-plan.md`

**Edit:**
- `contracts/stencils/rubric_test.go`
- `contracts/stencils/discussiontemplate_test.go`
- `contracts/stencils/plantemplate_test.go`

**Intent:**
Ask the attack-surface question where a decision is made and where a card states it (D12).

- Both rubrics gain an "Also flag" item: for every new verb, flag, escape hatch or routing edge, and every guard that is removed, downgraded or bypassable, the artifact states what it can now skip or let through and what bounds it; an unbounded one is a finding.
  The discussion rubric asks it of the decision record's Decisions; the plan rubric asks it of the card that introduces the change.
- The discussion writer stencil instructs the writer to state that bound in the Decision that introduces such a change, and the plan writer stencil instructs the writer to state it in the introducing card's `**Intent:**`, keeping the writer/reviewer symmetry each rubric already invokes.

Tests: `rubric_test.go` asserts both rubrics carry the item (the phrase "skip or let through" and "what bounds it"); `discussiontemplate_test.go` and `plantemplate_test.go` assert each writer stencil carries the matching instruction.

**ImpactSummary:** stencil-test assertions only; the four stencils change what the discussion and plan writers and reviewers read, and no Go code reads their text.
