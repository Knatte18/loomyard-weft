# Webster kick-start pack: per-card pack and pack-on read texts

Batch 01 (`01-pack-var-const-first-line`) ran as one fork and recorded `done` at head `00c5d027d`.
The batch carried the plan's three cards, committed one per card:

- `2b313836b`: a `var` or `const` pack entry shows the first line of its signature only.
- `70d884eac`: the pack is resolved once per batch and grouped per card.
- `00c5d027d`: step 1 and the FRESH-READ rule become markers with pack-off and pack-on fragments.

The fork reported its per-card gates and the batch gate green.
It reported one plan defect: cards 2 and 3 and the batch gate name `./internal/websterengine_test`, which has no directory of its own, so it ran the gates on `./internal/websterengine`.
That path needs fixing in the plan.

`record-batch` returned no deviations, only informational scope warnings.
It flagged `prePackStepOne`, `prePackFreshRead` and `writePrePackBody` in `internal/websterengine/template_test.go`, and `Index` in `internal/planindex/planindex.go`, as touched outside the batch's own target glyphs.
