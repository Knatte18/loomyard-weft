# PR-Rework stencil calls agent findings the operator's

`rework-findings-not-operator` · Note · bug, stencil, loom

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F6); line numbers as of `main` on 2026-10-16.

`contracts/stencils/loom/loom-template-rework.md:12` and `:29` tell the PR-Rework writer that the rejection's findings are "the operator's findings".
A PR rejection comes from the alignment agent, whose report becomes the findings (`internal/landingshed/pralign.go`, `applyReject`), or from the orch's reviewer fork; `contracts/stencils/orch/orch-template-role.md` has the orch, never the operator, decide at PR-Gate.
A writer told the findings are the operator's may treat an agent's claim as an operator decision.

## Decision

Operator decision (2026-10-16): the wording is wrong and is to be fixed.

## Idea

Agent's suggestion: call them the reviewer's findings on the rejected pull request, and say they come from the alignment agent or the parent, so the writer weighs them as a review rather than as an operator ruling.
