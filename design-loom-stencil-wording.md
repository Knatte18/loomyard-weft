# Loom and shed stencil wording fixes from the stencil intent audit

`loom-stencil-wording` · Note · bug, stencil, loom, shed

Bundle of the loom and shed wording findings of the stencil intent audit of 2026-10-15.
Line numbers as of `main` on 2026-10-16.

## Rejection findings are a reviewer's, not the operator's (F6, was `rework-findings-not-operator`)

### Seen

`contracts/stencils/loom/loom-template-rework.md:12` and `:29` tell the PR-Rework writer that the rejection's findings are "the operator's findings".
A PR rejection comes from the alignment agent, whose report becomes the findings (`internal/landingshed/pralign.go`, `applyReject`), or from the orch's reviewer fork; `contracts/stencils/orch/orch-template-role.md` has the orch, never the operator, decide at PR-Gate.
A writer told the findings are the operator's may treat an agent's claim as an operator decision.
`contracts/stencils/loom/loom-rubric-plan-review.md:82-83` makes a rework generation's `findings.md` part of Plan-Review's answer key and lets a finding license a card's scope, so the same agent's claim is weighed as a decision there too.

### Decision

Operator decision (2026-10-16): the wording is wrong and is to be fixed.

### Idea

Agent's suggestions:

- Call them the reviewer's findings on the rejected pull request, and say they come from the alignment agent or the parent, so the writer weighs them as a review rather than as an operator ruling.
- In `loom-rubric-plan-review.md:82-83`, say the rejection findings are a reviewer's claims the rework plan answers, not an operator ruling on a par with the decision record.

## Three small wording defects (F15 to F17, was `stencil-wording-fixes`)

### Seen

- F15: `contracts/stencils/shed/shed-template-driver.md:67` says a fix committed by hand outside the run is pushed by the operator before `approve`; nobody commits by hand in a task pair, and `orch/orch-template-role.md:99` forbids the orch to.
- F16: `contracts/stencils/loom/loom-template-discussion-advisor.md:10` has the advisor read the decision record "for its format", but the record does not exist until the chair writes it, and the advisor never writes one.
- F17: `contracts/stencils/loom/loom-template-plan.md:15` says the Plan producer "never asks", while it has two parent requests: `:35` has it ask its parent to record a late scope addition, and `:40` to record the drop of a decided step.
- Same kind as F17: `contracts/stencils/loom/loom-template-rework.md:14` says "You never interview and never ask" beside `{{.parent_directive}}`, whose text makes the parent "the one to ask" (`parent-directive-parent.md:7`).

### Decision

Operator decision (2026-10-16): F15 to F17 are to be fixed as the agent proposed.
Operator decision (2026-10-16): the same-kind `loom-template-rework.md:14` gap is fixed with them.

### Idea

Agent's suggestions:

- F15: strike the sentence part; a fix to a halted run goes through `reject` or a `goto`, so the run's own agents make it.
- F16: strike "for its format"; the advisor reads the draft record once it exists, to check it against PATTERN and the board, as `:46` already says.
- F17: make `:15` say it never asks the operator, and that its only messages are the two requests to its parent at `:35` and `:40`.
- `loom-template-rework.md:14`: say the rework writer never asks the operator, and asks its parent as the parent directive says.
