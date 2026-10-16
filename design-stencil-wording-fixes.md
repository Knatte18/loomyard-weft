# Three small stencil wording fixes from the intent audit

`stencil-wording-fixes` · Note · bug, stencil, shed, loom

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F15 to F17); line numbers as of `main` on 2026-10-16.
Three small wording defects:

- F15: `contracts/stencils/shed/shed-template-driver.md:67` says a fix committed by hand outside the run is pushed by the operator before `approve`; nobody commits by hand in a task pair, and `orch/orch-template-role.md` forbids the orch to.
- F16: `contracts/stencils/loom/loom-template-discussion-advisor.md:10` has the advisor read the decision record "for its format", but the record does not exist until the chair writes it, and the advisor never writes one.
- F17: `contracts/stencils/loom/loom-template-plan.md:15` says the Plan producer "never asks", while `:35` has it ask its parent to record a late scope addition.

## Decision

Operator decision (2026-10-16): all three are to be fixed as the agent proposed.

## Idea

- F15: strike the sentence part; a fix to a halted run goes through `reject` or a `goto`, so the run's own agents make it.
- F16: strike "for its format"; the advisor reads the draft record once it exists, to check it against PATTERN and the board, as `:46` already says.
- F17: make `:15` say it never asks the operator, and that its one message is the request to its parent in Step 1.
