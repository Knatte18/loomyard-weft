---
round: 1
ledger:
  - key: probe-error-degrades-auto
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: openlaunch-workspace-write-refusal-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: probe-limits-undocumented
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
F1 (`probe-error-degrades-auto`) says every probe error under `auto` drops the sandbox, including timeouts and a missing binary; its fix introduces a distinguishable error kind, so it changes a contract.
F2 (`openlaunch-workspace-write-refusal-untested`) asks for a test row pinning `OpenLaunch`'s workspace-write refusal and the probe arguments.
F3 (`probe-limits-undocumented`) asks for one doc-comment sentence on the probe's limits.
