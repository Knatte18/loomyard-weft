---
round: 2
ledger:
  - key: d12-crash-guard-findrun-premise
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d3-one-batch-partition-starves-shared-checks
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: active-implementer-undefined-d1-guard-late
    rounds: [1]
    status: resolved
  - key: renumber-bound-loses-state-lease
    rounds: [1]
    status: resolved
  - key: decision-notice-held-back-from-reattached-strand
    rounds: [1]
    status: resolved
  - key: describe-and-rework-archive-read-webster-only
    rounds: [1]
    status: resolved
  - key: d11-defers-shared-pieces-move
    rounds: [1]
    status: resolved
  - key: failed-attempt-strand-never-stopped
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-way-forward-names-missing-reset-verb
    rounds: [1]
    status: resolved
  - key: reentry-bound-names-wrong-caps
    rounds: [1]
    status: resolved
  - key: weaver-fixer-runs-under-webster-recovery-timeout
    rounds: [1]
    status: resolved
  - key: merge-in-inflight-warning-misses-weaver
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: journal-double-append-on-crash
    rounds: [1]
    status: resolved
  - key: same-rung-retry-scope-unstated
    rounds: [1]
    status: resolved
  - key: scope-out-push-contradicts-open-risk
    rounds: [1, 2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: stale-support-log-q6-q9-q10
    rounds: [1]
    status: resolved
  - key: verify-fixer-has-no-crash-identity
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reset-leaves-plan-and-records-in-weft
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reset-bound-ignores-live-strand
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-no-spawn-bypasses-row-gates
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: needs-parent-stop-reenters-as-ladder-stop
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reentry-has-no-weaver-rebaseline
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: one-producer-reading-of-operator-decision
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: stop-then-record-turns-crash-into-machinery-loss
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: corrupt-state-move-aside-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: decisions-delivered-twice-to-attached-strand
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-key-list-omits-verify-gate
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d11-carries-signature-detail
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d11-geometry-single-directory
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised sixteen keys.
Round 2 does not repeat eleven of them, so they are carried as resolved.
Five reopen in round 2:

- the D12 crash guard, now a BLOCKING re-entry-order defect (F1);
- the batch key and batch list for the shared checks (F2);
- strands that end any way but failure or timeout (F3);
- the merge-in in-flight warning, now lacking a Weaver done marker (F12);
- who pushes the journal mid-row, a `decision` finding for the parent (F10).

New design keys in round 2 cover the verify fixer's crash identity (F4), the reset's weft and live-strand bounds (F5, F6),
the D7 re-entry path (F7, F8, F9), the stop-then-record order (F13) and the corrupt-state bypass (F14).
F11 is a LOW `decision` finding for the parent on reading the operator's "each batch is a producer".
F15 to F18 are mechanical consistency fixes.
