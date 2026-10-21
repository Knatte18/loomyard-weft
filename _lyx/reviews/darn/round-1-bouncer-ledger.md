---
round: 1
ledger:
  - key: orch-rules-override-left-in-place
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: boardengine-intake-docs-stale
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: hubrepo-closure-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: import-close-unresolved-repo-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: neither-source-error-text-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orchcli-doc-friction-intake-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none BLOCKING.
Two are gating: the stale boardengine intake documentation and the untested `hubRepo` closure.
The orch-rules override finding is an unrecorded parent correction, deferred to the parent.
The remaining three are LOW test and doc gaps.
