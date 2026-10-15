---
round: 1
ledger:
  - key: work-contains-out-shadowed-by-overlap
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: pack-check-code-row-hits-records-refusal-first
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: default-out-fails-without-scratch-dir
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: single-spelling-rule-too-broad
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: report-header-omits-records-and-code-paths
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: marker-row-obligation-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings (F1 to F6), mapped in order to the keys above.
Two are BLOCKING refusal-reachability defects in cards 1 and 5; two are MEDIUM (default `--out` on a worktree without `.scratch/`, and an over-broad literal-spelling rule); two are LOW scope gaps.
None is `design` class, and every fix is a local text correction the finding already specifies.
