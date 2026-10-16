# Orch leaves comparing related runs to the alignment agent

`orch-no-peer-compare` · Note · bug, stencil, orch

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F9); line numbers as of `main` on 2026-10-16.

`contracts/stencils/orch/orch-template-role.md:94` has the orch read both Discussions itself at parent-review for related runs, pause both after Plan-Write to compare their plans, and review both PRs side by side.
That predates the alignment agent: `:40-45` of the same file has the alignment agent settle parent-review, an escalated review go to a one-shot fork, and the orch's context grow by the fork's summary only.
The alignment agent checks overlap with every run in flight and with the parent's landed commits, at both parent-review and PR-Gate, and escalates a real design conflict (`loom/loom-template-alignment.md:36`, `:55-71`).

## Decision

Operator decision (2026-10-16): comparing related runs is the alignment agent's job; the orch does not do it itself.

## Idea

Agent's suggestion: drop the reading, pausing and comparing in `:94`, and keep "two runs never talk to each other; you align them" only for an alignment escalation that names a peer, handled with the peer's escalation as `:68` already says, by a one-shot fork that reads both decision records.
Plans are not compared directly, since they follow from the records alignment compared, and a code-level clash surfaces at Merge-In and Merge-Review.
