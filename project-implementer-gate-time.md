# P8 Webster and Weaver spend less of their row in gate runs

Rendered 2026-11-28 15:21 UTC; every age below is as of then.
Under: [Board](README.md).

[`implementer-gate-time`](design-implementer-gate-time.md) · Age 3h · Owner `neve`

## Aim (operator, 2026-11-28)

The implementer row spends much of its time in gate runs: card gates, batch gates, the plan-level verify and the slot waits between them.
This project cuts how often and how widely the implementers gate, where project `test-speed` makes the tests themselves faster.

## Parts

1. A measurement of where the implementer rows' gate time goes now.
   Done: `docs/research/webster/gate-time-2026-11-28.md`.
2. Cuts filed from its report, one task per lever the operator picks.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **The batch gate runs tier1 over the batch own packages, not their dependents** | [`batch-gate-tier1-own-packages`](design-batch-gate-tier1-own-packages.md) | darn · light | landing · Finalize | neve | 1h<br>1h | gate, webster |

## Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **The card gate runs over the card own packages and its fixer stops at the failed check** | [`card-gate-own-packages`](design-card-gate-own-packages.md) | darn · light | T6 | 3h<br>50m | gate, webster, weaver |

## Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E6 |  | **An implementer batch gets the files and symbols its cards name, so it starts editing sooner** | [`implementer-orientation-read`](design-implementer-orientation-read.md) | 3h<br>1h | webster, weaver, stencil |
