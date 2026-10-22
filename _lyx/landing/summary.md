# The twill-plan fan runs five lenses instead of eight

The Plan writer's `twill-plan` fan spawned eight lens forks, each inheriting the Plan lead's large context.
A lens-overlap audit found several of them redundant, so each plan paid for forks that repeated each other's findings.
The fan now runs `coverage`, `row-validity`, `forced-edits`, `guard-reach` and `way-forward`, in that order.
The change is in the embedded lenses template, so every hub gets it.

- `coverage` merges `criteria-coverage` and `branch-coverage`.
- `row-validity` merges `row-reachability`, `test-economy` and the zero-value or nil seam check that `affected-set` carried.
- `forced-edits` is rewritten to absorb `affected-set`'s set method: callers, asserting tests, docs, skills and fixtures, and one finding per partly covered set.
- `guard-reach` and `way-forward` are unchanged; each raised BLOCKING findings no other lens did.

## Design decisions

- `criteria-coverage`, `branch-coverage`, `row-reachability` and `test-economy` are deleted from the template, since no fan runs them any longer.
  A hub `lenses.yaml` that still names one of them as a fan member gets an undefined-lens refusal at fan resolution; a hub-only definition of such a lens is kept and reported by reconcile.
- `affected-set` stays defined because `review-plan` still runs it.
  `review-plan`, `twill-discussion`, the other review fans and the fixer fans are untouched, as are the language layers under `internal/lenses/sets/`.
- The merged texts keep the sentences `config_test.go` pins: "A branch no planned row reaches is a finding.", "uses the operand the record names, not a near one" and "a registration no card lists is a finding".
  Each opens with "Own", ends with "Your list:" and then the emphasis line, as the template requires of a lens only writer fans run.
- The perch-parity test (a writer fan runs every lens of its reviewer fan) now accepts `forced-edits` in place of `affected-set` for `review-plan`.
  The mapping names that one pair; any other reviewer lens must still appear in its writer fan by name.
- The `TestPlanTable` case that redefines `twill-plan` through `lenses.yaml` used `test-economy`, which no longer exists; it uses `generic` and no longer needs an exemption in its dropped-lens check.
- The measurement reports under `docs/research/` still use the old lens names, as they record past runs.

## Tests

`config_test.go` pins the new fan order, the lens and fan counts, the perch-parity mapping and the sentences above.
`go test` was not run by hand here.
