---
round: 3
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
    status: resolved
  - key: expect-session-mark-removed-with-runs-json
    rounds: [2]
    status: resolved
  - key: diff-key-patterns-not-frozen-per-run
    rounds: [2]
    status: resolved
  - key: command-mode-validates-missing-identities
    rounds: [2]
    status: resolved
  - key: command-mode-resume-rerolls-ruling
    rounds: [2]
    status: resolved
  - key: judge-snapshot-misses-ignored-and-weft
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-judgement-yaml-passes-entry
    rounds: [2]
    status: resolved
  - key: judge-guard-stuck-no-way-forward
    rounds: [2]
    status: resolved
  - key: stucks-without-way-forward
    rounds: [2]
    status: resolved
  - key: capture-lock-refuses-no-lock-order
    rounds: [2]
    status: resolved
  - key: live-old-agents-left-running
    rounds: [2]
    status: resolved
  - key: waive-removal-strands-inflight-runs
    rounds: [2]
    status: resolved
  - key: partial-skip-field-removed-but-read
    rounds: [2]
    status: resolved
  - key: partial-flag-skip-list-pruned-first
    rounds: [2]
    status: resolved
  - key: unsealed-run-sealed-later-by-writer
    rounds: [2]
    status: resolved
  - key: expect-seals-over-moved-warp
    rounds: [2]
    status: resolved
  - key: d4-only-rule-contradicts-deferral-and-off
    rounds: [2]
    status: resolved
  - key: d8-silent-on-missing-baseline
    rounds: [2]
    status: resolved
  - key: drift-verify-gate-prune-unproven
    rounds: [2]
    status: resolved
  - key: d8-rationale-names-key-d1-never-produces
    rounds: [2]
    status: resolved
  - key: d11-counts-two-lists-three
    rounds: [2]
    status: resolved
  - key: mutable-records-bypass-seal
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-baseline-change-names-conflict
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

# Ledger, round 3

Round 3 is a fresh review of three findings; it did not re-raise any key from round 1 or round 2 except one, so those are carried as resolved.

- Records bound: `mutable-records-bypass-seal` (BLOCKING, F1).
  `scope.json` and `judged.json` hold containment state in plain tracked files, and D6's Bound claims a forged seal needs a hand-made weft commit.
- Judge guard: `judge-snapshot-misses-ignored-and-weft` (F2) reopens with a narrowed scope.
  The weft half is covered; ignored paths, the snapshot's location under the fabric-wired `.lym`, a modified or deleted untracked file, and a lost snapshot remain.
- Local consistency: `d9-baseline-change-names-conflict` (F3), where D9's Decision and its acceptance bullet disagree on what a judge-made baseline change names.
