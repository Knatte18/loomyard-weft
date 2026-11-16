---
round: 1
ledger:
  - key: d12-crash-guard-findrun-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d3-one-batch-partition-starves-shared-checks
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: active-implementer-undefined-d1-guard-late
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: renumber-bound-loses-state-lease
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: decision-notice-held-back-from-reattached-strand
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: describe-and-rework-archive-read-webster-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d11-defers-shared-pieces-move
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: failed-attempt-strand-never-stopped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-way-forward-names-missing-reset-verb
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reentry-bound-names-wrong-caps
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: weaver-fixer-runs-under-webster-recovery-timeout
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: merge-in-inflight-warning-misses-weaver
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: journal-double-append-on-crash
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: same-rung-retry-scope-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-out-push-contradicts-open-risk
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-support-log-q6-q9-q10
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review.
Findings F1 to F16 map to the keys above in order.
Two BLOCKING design findings stand: the crash guard needs a strand identity fixed before the spawn, and the partition Weaver records must carry the pending cards for the shared checks.
Six MEDIUM design findings concern the active-implementer seam, the renumber lease, the decision notice, Describe and rework readers, the deferred state-shape question, and stopping failed strands.
The remaining findings are LOW or NIT.
