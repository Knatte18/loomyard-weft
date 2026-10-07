---
round: 1
ledger:
  - key: roles-map-needs-declared-open-map
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: ttl-resolve-point-implicit
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: start-guard-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: ttl-log-site-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: acceptance-misses-whole-set-and-value-shapes
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: reed-add-not-a-ttl-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 findings, mapped one to one: F1 to `roles-map-needs-declared-open-map`, F2 to `ttl-resolve-point-implicit`, F3 to `start-guard-bound-unstated`, F4 to `ttl-log-site-unstated`, F5 to `acceptance-misses-whole-set-and-value-shapes`, F6 to `reed-add-not-a-ttl-path`.
All are open; the two design-class entries gate.
