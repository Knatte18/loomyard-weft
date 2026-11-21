---
round: 1
ledger:
  - key: hold-send-failed-title-literal
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: failed-hold-line-send-never-retried
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-findings-named-by-later-non-gate-stuck
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: final-reruns-check-on-held-tree
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-comments-kept-findings-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: uncovered-red-first-hold-transitions
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: preflight-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, all open.
F1 (title literal) is BLOCKING under PATTERN-code-conventions.
F2 (hold line sent once) and F3 (stale findings named by a later Stuck) are the gating `design` findings.
F4 to F7 are LOW: a repeated check on the held tree, four stale comments, uncovered hold transitions, and an untested Preflight call site.
