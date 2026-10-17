# Drop the per-refusal table from refusal-spec.md, keep the way-forward rule

`refusal-spec-table-drop` · Task · enhancement, shed, loom, webster, stencil · running · Darn-Write

## Seen (2026-10-17)

`contracts/specs/refusal-spec.md` was added on 2026-10-01 (`9ae5f9405`, "Every webster, shed and loom refusal names a way forward") and has had 81 commits since.
It is 510 lines and 133 KB: lines 1-34 define a way forward and the refusal classes (about 3 KB); the rest is 428 table rows, one per refusal, in sections webster, shed, loom, gate, batten lock, pause and hibernate.
`PATTERN-refusal-way-forward` requires every new refusal in a bound module to land with its row in the same commit, so the file grows with every task that adds a refusal.
Each row's Way forward cell quotes the `way forward:` clause the message itself ends with, so the table duplicates text the code carries (577 `way forward: ` clauses under `internal/`).
Readers: `shed-template-driver.md` (the LLM driver; the hub now runs the Go driver), `loom-template-plan.md` and `loom-rubric-plan-review.md` (for the grouped-transient and fail-loud-guard exemptions), `citation_enforcement_test.go`, several `doc.go` files and `docs/overview.md`.

## Operator view, not a decision (2026-10-17)

The operator does not remember the table as part of the design; it reads as a growing reference work rather than something an agent needs.

## Operator decision (2026-10-17)

The per-refusal table is removed, as the scope below says; the task runs as darn with no review rounds.

## Scope

Keep the rule that every refusal message ends in a `way forward:` clause, and keep the definition and classes (lines 1-34) as the spec.
Drop the per-refusal table and the PATTERN clause that requires a row per refusal; the message and the test that reaches it stay the record.
Move the two exemptions the Plan stencils cite (grouped transient I/O row, fail-loud wiring guard) into the definition part so the stencils keep a target, and update the citation test, the driver stencil pointer and the doc.go mentions.
