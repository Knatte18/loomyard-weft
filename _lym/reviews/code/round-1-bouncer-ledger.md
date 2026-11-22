---
round: 1
ledger:
  - key: unset-breaks-reconcile-only-rule
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hubreconcile-doc-names-reconcile-as-one-removal
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: refuse-raw-fallback-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: marked-hubs-never-get-scopeless-removal
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: invalid-refuse-raw-refusal-names-wrong-key
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unset-load-failure-worded-as-set
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-as-proxy-for-failing-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unset-rows-miss-file-atomicity-absent
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-unset-refusals-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: flagged-way-forward-under-asserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: contained-test-kept-without-mark
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: static-help-prose-asserted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: blast-radius-names-less-than-diff
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: repeated-unset-key-refuses-write
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised fourteen findings, none carried from an earlier round.
The gating ones are the BLOCKING PATTERN conflict (F1) and the untested launch fallback (F3).
F1 needs a rule-text change that a fresh review should see, as does F4, which waits on the parent's re-keying call.
The rest are local corrections the review text fixes.
