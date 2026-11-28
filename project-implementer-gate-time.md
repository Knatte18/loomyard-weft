# P8 Webster and Weaver spend less of their row in gate runs

Rendered 2026-11-28 12:02 UTC; every age below is as of then.
Under: [Board](README.md).

[`implementer-gate-time`](design-implementer-gate-time.md) · Age 2m · Owner `neve`

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
| T9 | **Measure where Webster and Weaver rows spend their gate time since the card gate took in dependent packages** | [`implementer-gate-time-measure`](design-implementer-gate-time-measure.md) | darn · docs |  | 2m<br>2m | gate, webster, weaver |
