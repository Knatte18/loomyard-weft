---
round: 2
ledger:
  - key: interrupt-settle-no-run-to-resolve
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: webster-default-engine-unbounded
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: attach-mismatch-skips-requires
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: r2-misstates-changed-default
    rounds: [1]
    status: resolved
  - key: instruction-files-no-scenario
    rounds: [1]
    status: resolved
  - key: attach-refusal-way-forward-unreachable
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
---

Round 1's five findings: two are gone, three narrowed.
`r2-misstates-changed-default` and `instruction-files-no-scenario` drew no round-2 finding and are resolved.
`interrupt-settle-no-run-to-resolve` narrowed to F4: engine resolution works for every caller, but the failure behaviour of a settle wait is unspecified.
`webster-default-engine-unbounded` narrowed to F3: the bound omits a changed `shuttle.yaml` engine default as a second divergence condition.
`attach-mismatch-skips-requires` reopened as F1: the attach `Requires` check reads forks from the spec, while the record decides the fork audit.
New in round 2 is F2: an attach refusal names a way forward that cannot clear it.
