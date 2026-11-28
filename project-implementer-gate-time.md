# P8 Webster and Weaver spend less of their row in gate runs

Rendered 2026-11-28 14:26 UTC; every age below is as of then.
Under: [Board](README.md).

[`implementer-gate-time`](design-implementer-gate-time.md) · Age 2h · Owner `neve`

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
| T6 | **The batch gate runs tier1 over the batch own packages, not their dependents** | [`batch-gate-tier1-own-packages`](design-batch-gate-tier1-own-packages.md) | darn · light | Code-Burler-Fix | neve | 1h<br>1h | gate, webster |

## Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E4 |  | **The card gate leaves the guard step to the batch gate** | [`card-gate-no-guard-step`](design-card-gate-no-guard-step.md) | 2h<br>1h | gate, webster, weaver |
| E6 |  | **An implementer batch gets the files and symbols its cards name, so it starts editing sooner** | [`implementer-orientation-read`](design-implementer-orientation-read.md) | 2h<br>1h | webster, weaver, stencil |
| E7 |  | **The last batch gate covers cmd/lym and cross-package tests, so the plan-level verify seldom fails** | [`batch-gate-covers-cmd-lym`](design-batch-gate-covers-cmd-lym.md) | 2h<br>1h | gate, webster, weaver |
