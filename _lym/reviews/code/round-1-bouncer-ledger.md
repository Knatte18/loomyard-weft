---
round: 1
ledger:
  - key: producer-engine-choice-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: codex-fork-wait-per-fork-wording
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: master-stencil-sendmessage-outside-block
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: stencil-headers-subagent-model-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loom-yaml-subagent-model-description
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: missing-copy-check-skips-parse-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darninputs-subagentmodel-doc
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, all open.
F1 is the only gating one: no test row tells engines apart for the producers that now pick an engine for their prompt.
The rest are wording and consistency leftovers from moving Claude-only text into engine blocks, plus one silent skip in a test helper.
