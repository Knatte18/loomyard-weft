---
round: 1
ledger:
  - key: long-cap-unbudgeted-cmd-lyx-unverified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: validate-machine-covered-unit-refusal-wording
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card1-machine-route-bound-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: gate-local-file-failure-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pattern-config-layers-four-categories-count
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: machine-loadcheck-refusal-stale-removal-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none recurring.
F1 gates: card 3's `--layer` gloss would push `lyx config` Long text past its cap, and verify runs no `cmd/lyx` test.
F2 and F3 are MEDIUM non-gating (refusal wording; missing bound on the machine route).
F4 is a LOW design finding about the local-file way forward; its fix changes what `ConfigFiles` returns, so it needs a fresh review.
F5 and F6 are LOW consistency findings with local fixes.
