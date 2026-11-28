# Webster and Weaver spend less of their row in gate runs

`implementer-gate-time` · Project · enhancement, gate, webster, weaver

## Aim (operator, 2026-11-28)

The implementer row spends much of its time in gate runs: card gates, batch gates, the plan-level verify and the slot waits between them.
This project cuts how often and how widely the implementers gate, where project `test-speed` makes the tests themselves faster.

## Parts

1. A measurement of where the implementer rows' gate time goes now.
2. Cuts filed from its report, one task per lever the operator picks.
