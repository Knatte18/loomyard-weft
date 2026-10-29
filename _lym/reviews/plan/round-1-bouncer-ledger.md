---
round: 1
ledger:
  - key: wire-seam-tests-nested-pullrequest-deps
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: wait-never-rereads-continue-record
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d9-pause-replaced-without-decision
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: closed-pr-with-continue-skips-checks
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: resolver-github-host-test-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: continue-poll-breaks-once-a-minute-bound
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: shuttle-empty-outputfiles-precondition-falsified
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stencil-card-names-wrong-symbol
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: falsified-doc-text-unlisted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: prreview-gate-key-beyond-d3
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: pr-guard-misses-issue-endpoint-close
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: publish-deny-bound-under-reports
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; every finding is new.
Three BLOCKING findings gate: the nested `PullRequest` seams break the wire seam tests, `wait` never re-reads the record so a mid-wait continue is lost, and D9's pause step is replaced in-process with no recorded decision.
Design-class MEDIUM findings also gate: a closed pull request with continue skips the clean-tree and gated-head checks, the resolver's github.com host test is undefined, and falsified doc text is left off target lists.
The wait re-read, the closed-verdict rework and the D9 pause step change behavior or a decision, so they are `design-wrong` and need a fresh review.
The rest are local corrections the finding text fixes.
