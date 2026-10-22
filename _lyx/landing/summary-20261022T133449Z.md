# The twill-plan fan runs five lenses instead of eight

The Plan writer's `twill-plan` fan spawned eight lens forks, each inheriting the Plan lead's large context.
An overlap audit found `affected-set` and `forced-edits` near-duplicates, and `criteria-coverage`, `branch-coverage` and `row-reachability` overlapping heavily.
The fan now runs five lenses, in this order:

| Lens | Replaces |
| --- | --- |
| `coverage` | `criteria-coverage`, `branch-coverage` |
| `row-validity` | `row-reachability`, `test-economy`, the zero-value or nil seam check |
| `forced-edits` | `affected-set`, the old `forced-edits` |
| `guard-reach` | unchanged |
| `way-forward` | unchanged |

The change is in `internal/lenses/template.yaml`, so every hub that has not redefined `twill-plan` gets it.
No language layer under `internal/lenses/sets/` defines or orders these lenses.
`review-plan`, `twill-discussion` and every other review and fixer fan keep their lens lists.

## Lens texts

The three merged lenses are written from the audit's drafts, fitted to the template's style.
Each opens with "Own", states its procedure and ends with "Your list:", as `guard-reach` and `way-forward` do.
Because no review or fixer fan runs them, each still ends with the emphasis line, which the template test requires of every lens outside those fans.
`forced-edits` keeps the old text's "an external `_test` package included" detail, which the draft dropped.

## Design decisions

- `criteria-coverage`, `branch-coverage`, `row-reachability` and `test-economy` leave the template, since no fan runs them any longer.
  `affected-set` stays defined because `review-plan` runs it.
  A hub file whose own fan names a deleted lens would get an undefined-lens refusal; this hub's `lenses.yaml` names none.
- The template test's perch-parity check, that a writer fan runs every lens its reviewer fan judges it by, now counts `review-plan`'s `affected-set` as covered by `twill-plan`'s `forced-edits`, the lens that absorbed it.
  The mapping names that one pair, so any other reviewer lens still has to appear in its writer fan by name.
- The pinned-sentence check moves to the new lens names: `coverage` keeps "A branch no planned row reaches is a finding.", `forced-edits` keeps its registration sentence, and `row-validity` pins its nil-seam sentence.
- `internal/loomengine/plan_test.go`'s lens-review row that redefines `twill-plan` through `lenses.yaml` used `test-economy`; it now uses `generic`, which no template `twill-plan` lens shares, so the row's special-case exemption is gone.
- The measurement reports under `docs/research/` keep the old lens names, since they record past runs.
