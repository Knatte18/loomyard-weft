---
round: 1
ledger:
  - key: decision-add-prerun-eager-geometry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: creel-probe-pending-without-mark
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sandbox-s9-unquoted-sender
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hook-wait-unreachable-daemon-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shuttle-open-no-append-prompt-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five LOW findings: F1 maps to `decision-add-prerun-eager-geometry`, F2 to `creel-probe-pending-without-mark`, F3 to `sandbox-s9-unquoted-sender`, F4 to `hook-wait-unreachable-daemon-untested` and F5 to `shuttle-open-no-append-prompt-untested`.
None gates, so the loop converges.
