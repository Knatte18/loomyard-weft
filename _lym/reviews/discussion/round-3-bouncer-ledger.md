---
round: 3
ledger:
  - key: d12-crash-guard-findrun-premise
    rounds: [1, 2]
    status: resolved
  - key: d3-one-batch-partition-starves-shared-checks
    rounds: [1, 2]
    status: resolved
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
    rounds: [1, 2, 3]
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
    status: resolved
  - key: journal-double-append-on-crash
    rounds: [1]
    status: resolved
  - key: same-rung-retry-scope-unstated
    rounds: [1]
    status: resolved
  - key: scope-out-push-contradicts-open-risk
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stale-support-log-q6-q9-q10
    rounds: [1]
    status: resolved
  - key: verify-fixer-has-no-crash-identity
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: reset-leaves-plan-and-records-in-weft
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: reset-bound-ignores-live-strand
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-no-spawn-bypasses-row-gates
    rounds: [2]
    status: resolved
  - key: needs-parent-stop-reenters-as-ladder-stop
    rounds: [2]
    status: resolved
  - key: reentry-has-no-weaver-rebaseline
    rounds: [2]
    status: resolved
  - key: one-producer-reading-of-operator-decision
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stop-then-record-turns-crash-into-machinery-loss
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: corrupt-state-move-aside-unbounded
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: decisions-delivered-twice-to-attached-strand
    rounds: [2]
    status: resolved
  - key: d5-key-list-omits-verify-gate
    rounds: [2]
    status: resolved
  - key: d11-carries-signature-detail
    rounds: [2]
    status: resolved
  - key: d11-geometry-single-directory
    rounds: [2]
    status: resolved
---

Round 3 has seven findings, down from eighteen.
Its review rejects the crash re-entry order (`d12-crash-guard-findrun-premise`) and the batch identity and partition (`d3-one-batch-partition-starves-shared-checks`), so both are resolved.
Round 3 does not raise the merge-in warning, the D7 no-spawn path, the needs-parent re-entry, the Weaver rebaseline, the duplicate decision delivery or the D5 and D11 consistency keys, so they are resolved.

Still open, all on the reset, restore and strand-lifecycle edges:

- F1: the verify fixer outside `gatefixer`, BLOCKING, on `verify-fixer-has-no-crash-identity`.
- F2: the reset's live-strand check, on `reset-bound-ignores-live-strand`.
- F3: attach error and failed stop, on `failed-attempt-strand-never-stopped`.
- F4: the corrupt-state restore, on `corrupt-state-move-aside-unbounded`.
- F5: the two Open risks lines the added entries answer, on `scope-out-push-contradicts-open-risk` and `one-producer-reading-of-operator-decision`; both are now consistency fixes.
- F6: D6's failed-state-write sentence, on `stop-then-record-turns-crash-into-machinery-loss`.
- F7: the reset archive's destination and commit, on `reset-leaves-plan-and-records-in-weft`.
