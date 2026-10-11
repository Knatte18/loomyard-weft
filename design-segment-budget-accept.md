# Per-segment review budget and accept on a spent budget

`segment-budget-accept` · Note · enhancement, loom, burler

## Seen (2026-10-11)

`loom.yaml` `review_max_bounces` is one budget for every Bouncer review segment (Discussion-Review, Plan-Review, Webster-Review), and a spent budget always escalates to the parent, which today answers with `lyx loom circling accept`.
Nothing accepts on its own.
Webster-Burler's first rounds in the four newest runs yielded 0 BLOCKING, 1 MEDIUM, 1 LOW, 3 NIT and 2 empty reviews (`burler-chair-late-spawn`).

## Idea (operator, 2026-10-11)

For Go code the tests in verify carry the weight, so Webster-Review runs at most one round and then accepts the work as it stands, without escalating to the parent.
This needs two settings per segment: its own bounce budget (such as `webster_max_bounces`), and what a spent budget does (escalate, as today, or accept).

## Open

- After one round nobody checks the fixer's changes except verify; that holds for Go code, less for prose such as stencils (see `burler-chair-late-spawn`).
