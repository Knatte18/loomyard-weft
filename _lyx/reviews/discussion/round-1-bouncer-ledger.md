---
round: 1
ledger:
  - key: sandbox-list-vs-unchanged-bytes
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-forkaudit-dependency-no-fallback
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-bound-omits-trace-write
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: start-here-confound-in-open-risks
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, all still open.
The sandbox-list finding is the blocker: adding `lyx loom card-pack` to the shared body's allowed list conflicts with D4's unchanged bytes for the file, inline, pack-off and recovery prompts.
The D7 finding is that the decision rests on a `tools/forkaudit` that has not landed, and the record gives the plan writer no step when it is absent.
The D3 bound and the Open risks confound note are local text corrections.
