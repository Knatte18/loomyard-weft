# Decision record — refusal-spec-table-drop

## Goal

Stop `contracts/specs/refusal-spec.md` growing with every new refusal by dropping its per-refusal table, while keeping the rule that every refusal names its way forward.

## Scope

- `contracts/specs/refusal-spec.md` keeps the way-forward definition and the classes, and loses every per-module section, its preamble and table, and the `## Out of scope` section.
- The two exemptions the Plan stencils cite, the raw I/O transient and the fail-loud wiring guard, move into the kept part as a statement of which refusals need a reaching test.
- `PATTERN.md` and `pattern/PATTERN-refusal-way-forward.md` drop the row-per-refusal clause and name the bound modules directly instead of by their section in the spec.
- Readers update: the shed driver stencil and its guide, `loom-template-plan.md`, `loom-rubric-plan-review.md`, `citation_enforcement_test.go`, `webster-spec.md`, the `doc.go` mentions, `docs/overview.md`, `pattern/PATTERN-no-denied-recovery.md` and the comment in `cmd/lyx/norecoverydeny_test.go`.
- Out of scope: any refusal message or its `way forward:` clause in Go code.

## Decisions

### D1: The spec keeps the definition and classes only

Decision: the file keeps `## What a way forward is` and `## Classes`, reworded so no sentence refers to a row, a cell or a section, and gains a `## Reaching tests` section.
Its status note calls it the contract every refusal message follows rather than a lookup table.
Rationale: the message and the test that reaches it are the record, and the table quoted text the code already carries.

### D2: The bound modules are named, not derived from spec sections

Decision: `PATTERN-refusal-way-forward` binds webster, shed and loom by name; another module joins by a PATTERN change.
The no-denied-recovery background and the scan's comment say "the modules `PATTERN-refusal-way-forward` binds".
Rationale: with no sections left, "the modules with a section" names nothing.
Bound: the bound set stays webster, shed and loom, the set the scan already covers, so no guard narrows.

### D3: The driver reads the way forward from the message

Decision: the shed driver stencil tells the driver to quote the refusal's own `way forward:` clause beside it, and to report a refusal without one verbatim; it no longer points at `refusal-spec.md`, and the citation allowlist entry for that pointer goes.
Rationale: the clause is on the message the driver already holds, so a table lookup adds nothing.

### D4: The Plan stencils cite the moved exemptions

Decision: `loom-template-plan.md` and `loom-rubric-plan-review.md` cite the raw I/O transient and the fail-loud wiring guard as `refusal-spec.md` now states them, with no "grouped row" wording.
Rationale: the stencils keep a target that exists.

## Constraints

- `PATTERN-refusal-way-forward` changes in the same commit as the spec.
- `PATTERN-no-denied-recovery`: the reworded stencils and spec name no denied command.
- Semantic line breaks in every edited markdown file and Go comment.

## Auto-mode assumptions

- The `## Out of scope` section is dropped with the table, since it lists what the table covered.
- The webster section's preamble on fork-audit severity is dropped, since `internal/websterengine`'s `doc.go` already carries that classification.

## Open risks

- A deployed driver stencil from before this change still points at the table; it reads the file at its path in the loomyard worktree, which still exists, and finds no rows, so it reports refusals verbatim until the next deploy.

## Acceptance criteria

- `contracts/specs/refusal-spec.md` holds no refusal table and no per-module section.
- No file outside `docs/research/` refers to a refusal-spec row, table or section.
- `contracts/stencils` tests, the no-denied-recovery scan and the markdown link check pass.

## PATTERN changes

- `PATTERN-refusal-way-forward` — changes: Adding a webster, shed or loom refusal: its message names the way forward `contracts/specs/refusal-spec.md` defines, and a test reaches it.
