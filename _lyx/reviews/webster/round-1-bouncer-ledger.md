---
round: 1
ledger:
  - key: burlerengine-doc-fan-keys-names-burler-yaml
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: validatefankeys-doc-per-name-template-fallback
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW doc-wording findings.
F1: the `burlerengine` doc paragraph on `FanList` and `ValidateFanKeys` still names `burler.yaml` and the wrong package.
F2: the `ValidateFanKeys` doc keeps the retired per-name template fallback wording.
Neither is gating.
