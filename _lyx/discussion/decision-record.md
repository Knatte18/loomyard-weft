# Decision record — twill-plan runs 5 lenses instead of 8

## Goal

The Plan writer's `twill-plan` fan spawns five lens forks instead of eight, merging the lenses the overlap audit found redundant, so each plan pays for fewer forks of the lead's large context without losing a lens that raised unique BLOCKING findings.

## Scope

- In: the `twill-plan` fan and the lens definitions in `internal/lenses/template.yaml`, the tests in `internal/lenses/config_test.go` that pin them, and any test elsewhere that names a lens this change removes.
- In: the language layers under `internal/lenses/sets/`; none of them defines or orders a lens this change touches, so they stay as they are.
- Out: `review-plan`, `twill-discussion`, every other review fan and every fixer fan.
- Out: the measurement reports under `docs/research/`, which record past runs under the old lens names.

## Decisions

### D1: `twill-plan` runs `coverage`, `row-validity`, `forced-edits`, `guard-reach`, `way-forward`, in that order

Rationale: the operator's table, with the three merged lenses first since they carry most findings, and the two unchanged lenses in their current relative order.

### D2: three lens texts are written from the audit's drafts, fitted to the current texts

`coverage` merges `criteria-coverage` and `branch-coverage`.
`row-validity` merges `row-reachability`, `test-economy` and the zero-value or nil seam check.
`forced-edits` is rewritten to absorb `affected-set`'s set method: callers, asserting tests, docs, skills and fixtures, and one finding per partly covered set.
Each keeps the sentences `config_test.go` pins today where the merged lens carries the same check: "A branch no planned row reaches is a finding.", "uses the operand the record names, not a near one" and "a registration no card lists is a finding".
Each opens with "Own", ends with "Your list:", and then with the emphasis line, since no review or fixer fan runs it.
Rationale: the procedure shape matches `guard-reach` and `way-forward`; a lens only a writer fan runs ends with the emphasis line, as the template's own rule and its test require.

### D3: the four lenses no fan runs any longer are deleted

`criteria-coverage`, `branch-coverage`, `row-reachability` and `test-economy` leave the template.
`affected-set` stays defined because `review-plan` runs it.
Rationale: the template holds what its fans use; a hub file that still names a deleted lens keeps it as a hub-only lens, which reconcile keeps and reports.

### D4: the perch-parity test maps `affected-set` to `forced-edits`

The test that a writer fan runs every lens of its reviewer fan accepts, for `review-plan`'s `affected-set`, the writer lens that absorbed it, `forced-edits`.
Rationale: the operator decided both that `forced-edits` replaces `affected-set` in `twill-plan` and that `review-plan` is out of scope; the parity the test guards still holds through the absorbing lens.
Bound: the mapping names that one pair; any other reviewer lens still has to appear in its writer fan by name.

## Constraints

- `PATTERN-stencil-ownership`: lens prose stays in the embedded template the config module reads; no stencil restates it.
- No lens tells a fork to ignore anything.
- Semantic line breaks in every Go comment and markdown file.

## Auto-mode assumptions

- The audit's drafts are the content; wording is fitted to the template's rubric references and terminology without changing what each lens checks.
- `forced-edits` keeps the existing lens's "an external `_test` package included" detail, which the draft dropped.

## Open risks

- A hub whose own `lenses.yaml` fan names a deleted lens gets an undefined-lens refusal at fan resolution; no hub in this repository's board does.

## Acceptance criteria

- `ResolveFan(template, "twill-plan")` yields `coverage`, `row-validity`, `forced-edits`, `guard-reach`, `way-forward` in order.
- `review-plan`, `twill-discussion` and every other fan keep their current lens lists.
- The template defines `coverage` and `row-validity`, no longer defines the four deleted lenses, and still defines `affected-set`.
- Every lens a fan names is defined, and every lens only writer fans run ends with the emphasis line.
- The perch-parity check passes with `affected-set` covered by `forced-edits`, and still fails for any other reviewer lens the writer fan lacks.
- The loom plan-table test that redefines `twill-plan` through `lenses.yaml` still passes with a lens that remains defined.
