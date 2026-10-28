# Machine checks in validate-discussion and validate-plan for recurring review findings

`validator-checks` · Task · enhancement, loom, stencil · running · Loom-Preflight

## Seen

`docs/research/review-setup-audit-2026-10-28/report.md` (section 3) classed the gating Discussion and Plan findings and found themes a machine check could stop before a reviewer spends a round on them.
The operator agreed with its recommendations (2026-10-28); this task builds the checks.

## Design

`lym loom validate-discussion`, and the gates of the rows that run it:

- A citation check: every repository path and Go symbol the decision record cites exists.
- An acceptance check: every decision id appears in at least one acceptance criterion.

`lym loom validate-plan`, and the gates of the rows that run it:

- A card that adds a field to a shared `Deps` or `Env` struct lists the reflective seam tests of that struct.
- A card that adds a test file states exactly one tier tag for it.
- A card whose Intent changes a package's behaviour lists that package's `doc.go`.

Each check's refusal names the item it found and what the writer adds to fix it.
The Discussion and Plan writer stencils state the new rules, so a writer meets them before the gate does.

## Out of scope

- Lens and profile changes: a task of their own.
- Checks for the design-interaction and unbounded-edge classes, which the report leaves to the lenses.
