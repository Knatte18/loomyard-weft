---
round: 1
ledger:
  - key: card5-pushclassified-records-takeover
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: deferred-records-side-reads-as-landed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: branch-rewind-outside-destroy-chokepoint
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card8-spawn-field-tests-not-targets
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: teardown-wait-error-type-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card7-owner-guard-narrows-d2-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: card7-origin-reset-no-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card6-batten-record-read-via-child-reader
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: card9-end-only-cancel-premise-false
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: code-tip-way-forward-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card12-gives-up-locally-held-tip
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: detached-teardown-status-push-unnamed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: marktaskdone-resign-arrow-missing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fabriccli-doc-keeps-retired-respawn
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-pushlockbusy-owed-without-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card5-machinenames-second-construction-site
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 16 findings, mapped one to one onto the keys above in review order (F1 to F16); all are open.
The BLOCKING ones are the unrecorded-takeover claim in card 5, the deferred records side read as landed, branch rewinds outside `destroy.go`, and the five tests card 8 leaves uncompilable.
Design-wrong fixes (new decisions or contracts) need a fresh review next round; mechanical ones are local text or target edits.
