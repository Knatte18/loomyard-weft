---
round: 1
ledger:
  - key: comment-hard-wrap-mid-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: statuscommit-push-error-names-side
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: acquirewritelockwithin-dead-branch
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: push-helpers-untested-behaviors
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: loom-help-commit-records-records-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised five findings: F1 (BLOCKING, comment line breaks), F2 (statuscommit side-naming claim), F3 (dead lock branch), F4 (untested unborn-HEAD skip and second-push rejection), F5 (stale loom help text).
All are open; none has been seen by a fresh reviewer after a fix.
