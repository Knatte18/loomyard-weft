# One rule for a stuck review: the arbiter's, with the orch answering its question

`arbiter-filters-orch` · Note · bug, stencil, orch, loom

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F8); line numbers as of `main` on 2026-10-16.

A review loop that cannot settle goes to the arbiter first, and reaches the orch only when the arbiter hands on (`contracts/stencils/orch/orch-template-role.md:52`, `shed/shed-template-driver.md:63-64`).
The decision is worded three times:

- `loom/loom-template-arbiter.md:65-66` gives the arbiter its rule per cause (`hard-cap`, `circling`).
- `orch/orch-template-role.md:54` gives the orch its own: "accept unless a finding makes the design wrong, and hand a blocker that remains after round 3 to the operator", with a fixed round 3 that ignores the configurable hard cap.
- `bouncer/bouncer-template-escalation.md:31-39` has the orch's one-shot fork settle "on the materials" or report back to the orch, "which asks the operator".

Neither orch text accounts for the arbiter having already applied its rule.

## Decision

Operator decision (2026-10-16): the arbiter is the orch's filter.

## Idea

Agent's suggestions:

- The arbiter's per-cause rule is the only statement of how a stuck review is decided.
- What reaches the orch is the arbiter's hand-on question; the escalation brief has the fork answer that question from the operator's recorded decisions, and otherwise bring that question to the operator.
- `orch-template-role.md:54` and its fixed round 3 go, and the orch role points at the brief instead of restating a rule.
