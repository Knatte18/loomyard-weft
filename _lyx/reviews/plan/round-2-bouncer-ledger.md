---
round: 2
ledger:
  - key: test-budget-not-raised
    rounds: [1]
    status: resolved
  - key: drift-test-repeats-existing
    rounds: [1]
    status: resolved
  - key: reader-fallback-turn-end-untested
    rounds: [1]
    status: resolved
  - key: shuttleengine-doc-stale
    rounds: [1]
    status: resolved
  - key: claudeengine-doc-missing-parser
    rounds: [1]
    status: resolved
  - key: card-12-unlicensed-wording
    rounds: [1]
    status: resolved
  - key: output-files-first-no-carrier
    rounds: [1]
    status: resolved
  - key: card-13-doc-go-missing
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card-12-triage-test-missing
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: cards-10-11-wait-tests-repeat-existing
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card-11-expiredshells-comment-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card-2-deny-key-comments-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card-3-trailing-stamp-narrow
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: card-13-cursor-start-open
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: card-5-ask-exit-status-unpinned
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's seven findings do not reappear in round 2's fresh review, so they are carried as resolved.
Round 2 opened eight new findings, mapped by review ID: F1 `card-13-doc-go-missing`, F2 `card-12-triage-test-missing`, F3 `cards-10-11-wait-tests-repeat-existing`, F4 `card-11-expiredshells-comment-stale`, F5 `card-2-deny-key-comments-stale`, F6 `card-3-trailing-stamp-narrow`, F7 `card-13-cursor-start-open`, F8 `card-5-ask-exit-status-unpinned`.
None is design-class or BLOCKING, so the loop converged.
