# P8 Webster and Weaver spend less of their row in gate runs

Rendered 2026-11-28 12:06 UTC; every age below is as of then.
Under: [Board](README.md).

[`implementer-gate-time`](design-implementer-gate-time.md) · Age 6m · Owner `neve`

## Aim (operator, 2026-11-28)

The implementer row spends much of its time in gate runs: card gates, batch gates, the plan-level verify and the slot waits between them.
This project cuts how often and how widely the implementers gate, where project `test-speed` makes the tests themselves faster.

## Parts

1. A measurement of where the implementer rows' gate time goes now.
2. Cuts filed from its report, one task per lever the operator picks.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Ready (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **Measure where Webster and Weaver rows spend their gate time since the card gate took in dependent packages** | [`implementer-gate-time-measure`](design-implementer-gate-time-measure.md) | darn · docs |  | 6m<br>6m | gate, webster, weaver |

## Enhancements (4)

| # | Prio | Note | Slug | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| E4 |  | **The card gate leaves the guard step to the batch gate** | [`card-gate-no-guard-step`](design-card-gate-no-guard-step.md) | T9 | 0m<br>0m | gate, webster, weaver |
| E6 |  | **An implementer batch gets the files and symbols its cards name, so it starts editing sooner** | [`implementer-orientation-read`](design-implementer-orientation-read.md) | T9 | 0m<br>0m | webster, weaver, stencil |
| E7 |  | **The last batch gate covers cmd/lym and cross-package tests, so the plan-level verify seldom fails** | [`batch-gate-covers-cmd-lym`](design-batch-gate-covers-cmd-lym.md) | T9 | 0m<br>0m | gate, webster, weaver |
| E8 |  | **A gate run under the parent lock takes the next free gate slot ahead of runs that hold no lock** | [`lock-holder-gates-first`](design-lock-holder-gates-first.md) |  | 0m<br>0m | gate, landing |
