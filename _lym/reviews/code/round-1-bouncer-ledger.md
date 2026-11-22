---
round: 1
ledger:
  - key: clear-review-session-keeps-parent-review-asked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dobby-run-notices-unstamped-autonomy
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: halt-notice-call-sites-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: goto-hook-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: batten-goto-no-admitted-hook
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: pr-review-failure-paths-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: beam-await-drain-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: overview-slack-and-beam-await-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: with-launch-facts-testtiming-keep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hook-delivery-test-testtiming-keep
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; there is no prior ledger.
F1 (`clear-review-session-keeps-parent-review-asked`) is the one gating finding: a goto onto PR-Review leaves `ParentReviewAsked` set, so the first ask is never sent.
The other nine are test-coverage gaps, one missing autonomy stamp on dobby's run notices, a docs gap, and two `//testtiming:keep` marks; each fix is a local change its text already fixes.
