# Lens overlap audit: which lenses find the same defects, and a smaller set

`lens-overlap-audit` · Note · research, burler, loom

## Question (operator, 2026-10-22)

Which lenses find the same defects, which defect areas the lenses essentially cover, and whether a smaller lens set covers the same ground.

## Result (sonnet agent, 2026-10-22)

Sources: `_lyx/plan/lens-review.md` in the weft archive tags (94 loom runs, only 6 of them, all on 2026-10-22, with all 8 `twill-plan` lenses), the per-lens fork reports in the machine-local Claude Code transcripts (5 runs), and per-lens attribution in `round-1-review.md`.
The sample with the full lens set is small, lens texts changed over time, and a "fixed" finding only means the writer agreed; since `lens-fork-reports-kept` landed (79bab47a0) every lens fork's report is kept verbatim beside its record (`*.lens-<lens>.md` under `_lyx/plan/`, `_lyx/discussion/` and the review rounds), so later audits have per-lens data.

### `twill-plan` (8 lenses)

- `affected-set` and `forced-edits` are near-duplicates: 12 of `affected-set`'s 20 findings were also raised by `forced-edits`, 5 were its alone.
- `criteria-coverage`, `branch-coverage` and `row-reachability` overlap heavily; in one run all three reports led with the same three defects.
- `guard-reach` and `way-forward` overlap little with anything and each raised BLOCKING findings no other lens did.
- `test-economy` is 79% LOW and the most disputed, but its findings are unique.

Agent recommendation, not a decision: 5 lenses.

| Proposed lens | Replaces |
| --- | --- |
| `coverage` | `criteria-coverage`, `branch-coverage` |
| `row-validity` | `row-reachability`, `test-economy`, the nil-seam part of `affected-set` |
| `forced-edits` | `affected-set`, `forced-edits` |
| `guard-reach` | unchanged |
| `way-forward` | unchanged |

Merging `guard-reach` and `way-forward` as well is not supported by the data.

### `twill-discussion` (7 lenses), low confidence

Since 5e449651f `goal-scope` and `board-fit` are rewritten procedure lenses that also run in the `review-discussion` fan.

No lens-review file exists for the Discussion writer; one run's fork reports were read in full.
Agent recommendation, not a decision: 4 lenses, `decision-chain` (`interactions` + `walkthrough`, keeping the case-based procedure), `code-claims` (`bound-breaker` + `existing-paths`), `board-scope` (`goal-scope` + `board-fit`), `leanness`.
In the review fan, `bound-breaker` is the strongest lens (11 of 15 lens BLOCKING findings), `walkthrough` raised no BLOCKING.

### Review fans

- `review-plan` reruns the three lenses the Plan writer already ran, and still raised 35 findings in the six 2026-10-22 runs, mostly LOW and MEDIUM; whether these are bad fixes or new findings is not measured. Option: give `review-plan` the lenses the writer fan lacks.
- `review-code`: `code-affected-set` is the weakest (one MEDIUM in 18); agent recommendation, medium-low confidence: fold it into `code-pattern-fit`, which carries 7 of 8 lens BLOCKING findings, as one `code-consistency` lens, 4 to 3.
- `fix-*` fans: no per-lens data.

## Follow-up (operator, 2026-10-22)

- `twill-plan` goes to 5 lenses in task `twill-plan-five-lenses`.
- A later audit looks at the Discussion writer's `twill-discussion` alone, once enough runs made after 79bab47a0 have kept per-lens reports, and may trim it too.
  It matters less than Plan: the Discussion lead's context is far smaller when it forks, so its lens forks cost far less.
