---
round: 2
ledger:
  - key: partial-scope-no-reinterpretation
    rounds: [1]
    status: resolved
  - key: recorded-partial-scope-inflight
    rounds: [1]
    status: resolved
  - key: master-no-fixer-premise
    rounds: [1]
    status: resolved
  - key: post-writer-regression-no-way-forward
    rounds: [1]
    status: resolved
  - key: retired-keys-not-refused
    rounds: [1]
    status: resolved
  - key: command-mode-judge-picks-cases
    rounds: [1]
    status: resolved
  - key: seal-check-command-mode-contradiction
    rounds: [1]
    status: resolved
  - key: reentry-seals-interrupted-expect
    rounds: [1]
    status: resolved
  - key: restore-wrong-way-forward
    rounds: [1]
    status: resolved
  - key: judge-snapshot-misses-commit
    rounds: [1]
    status: resolved
  - key: past-drift-expect-test-unstated
    rounds: [1]
    status: resolved
  - key: amendment-reach-bound
    rounds: [1]
    status: resolved
  - key: gate-seam-overstated
    rounds: [1]
    status: resolved
  - key: drift-verify-gate-key-orphaned
    rounds: [1]
    status: resolved
  - key: past-recheck-lands-empty-verdict
    rounds: [1]
    status: resolved
  - key: passing-check-command-mode
    rounds: [1]
    status: resolved
  - key: d6-acceptance-hand-edited-record
    rounds: [1]
    status: resolved
  - key: open-range-statements-unread
    rounds: [1]
    status: resolved
  - key: baseline-lost-before-expect
    rounds: [1]
    status: resolved
  - key: identity-hash-sign-ambiguity
    rounds: [1]
    status: resolved
  - key: open-range-bound-missing
    rounds: [1]
    status: resolved
  - key: rationale-reargues-rejected
    rounds: [1]
    status: resolved
  - key: baseline-capture-unguarded
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: expect-session-mark-removed-with-runs-json
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: diff-key-patterns-not-frozen-per-run
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: command-mode-validates-missing-identities
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: command-mode-resume-rerolls-ruling
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: judge-snapshot-misses-ignored-and-weft
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-judgement-yaml-passes-entry
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: judge-guard-stuck-no-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stucks-without-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: capture-lock-refuses-no-lock-order
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: live-old-agents-left-running
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: waive-removal-strands-inflight-runs
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: partial-skip-field-removed-but-read
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: partial-flag-skip-list-pruned-first
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: unsealed-run-sealed-later-by-writer
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: expect-seals-over-moved-warp
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-only-rule-contradicts-deferral-and-off
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-silent-on-missing-baseline
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: drift-verify-gate-prune-unproven
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: d8-rationale-names-key-d1-never-produces
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d11-counts-two-lists-three
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

# Ledger, round 2

Round 2 is a fresh review: it did not re-raise any of round 1's keys, so all of them are carried as resolved, and its own findings are first-raised keys.

- Baseline and judge guard: `baseline-capture-unguarded` (BLOCKING), `judge-snapshot-misses-ignored-and-weft`, `stale-judgement-yaml-passes-entry`, `judge-guard-stuck-no-way-forward`.
- Frozen run state: `diff-key-patterns-not-frozen-per-run`, `expect-session-mark-removed-with-runs-json`, `partial-skip-field-removed-but-read`.
- Command mode: `command-mode-validates-missing-identities`, `command-mode-resume-rerolls-ruling`.
- Way forward and locks: `stucks-without-way-forward`, `capture-lock-refuses-no-lock-order`.
- In-flight runs: `live-old-agents-left-running`, `waive-removal-strands-inflight-runs`, `partial-flag-skip-list-pruned-first`, `unsealed-run-sealed-later-by-writer`, `expect-seals-over-moved-warp`.
- Local consistency: `d4-only-rule-contradicts-deferral-and-off`, `d8-silent-on-missing-baseline`, `drift-verify-gate-prune-unproven`, `d8-rationale-names-key-d1-never-produces`, `d11-counts-two-lists-three`.
