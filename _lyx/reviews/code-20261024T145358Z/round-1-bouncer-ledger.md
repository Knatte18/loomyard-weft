---
round: 1
ledger:
  - key: boardcli-doc-old-profiles-shape
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: upsert-help-omits-default-profile
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: docs-profile-content-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: profileless-task-undecodable-profiles-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: decodedefaults-comment-misleading
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five non-gating findings (F1 to F5, in the order above); none recurs from an earlier round.
All five are local corrections whose text fixes the result.
