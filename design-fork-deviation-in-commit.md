# A webster fork states a PATTERN departure in its commit message

`fork-deviation-in-commit` · Note · bug, stencil, webster, burler

## Seen

Found by the stencil intent audit (`docs/research/stencil-intent-audit-2026-10-15/findings.md`, F12 and F13); line numbers as of `main` on 2026-10-16.

A webster fork has no channel for two things:

- `contracts/stencils/webster/webster-body-implementer.md:98` has it name a failure its card did not cause "in your final reply to Master", but Master never reads raw fork output (`webster-template-master.md:101`, `:243-244`) and the batch report holds only `status` and `head_sha` (`webster-body-implementer.md:104`, which also says Master reads that report, when Go reads it through `record-batch`).
- `contracts/stencils/pattern/pattern-directive-implementer.md:19` has an implementer that meets a PATTERN conflict follow PATTERN and "say so in your report", which the same two-field report cannot hold.

A fork that follows PATTERN over its card leaves a diff that departs from the plan with no stated reason, so Webster-Review, which measures the diff against the plan, flags the departure and its fixer reverts it into the PATTERN violation.

## Decision

Operator decision (2026-10-16), on the agent's recommendation:

- A fork that departs from its card because of a PATTERN entry states the departure and the entry in that card's commit message body.
- Webster-Review's rubric reads the commit messages of its range as well as the diff, and takes a departure stated there as the reason, not as an unexplained mismatch.
- The "name it in your final reply to Master" sentence goes: a failure the card did not cause is left to the plan-level verify gate, which runs the whole suite and spawns a fixer.
- No new report field.

## Idea

Agent's suggestions: point `pattern-directive-implementer.md:19` at the commit message for a webster fork and at the plan's `## Shared Decisions` for the plan writer, its other reader; correct `webster-body-implementer.md:104` to say `record-batch` reads the two fields.
