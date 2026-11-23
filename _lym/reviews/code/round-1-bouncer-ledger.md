---
round: 1
ledger:
  - key: describer-stencil-restates-summary-format
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: describe-error-blocks-tidy-and-landing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unreadable-draft-keeps-attempt-in-flight
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: worktree-gone-leaves-describer-running
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pr-update-failure-misreported-as-unrecorded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merge-danger-called-two-way-for-outward-push
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: tick-wiring-of-describe-step-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-post-loses-failure-line
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: intermediate-draft-recorded-and-published
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: draft-write-truncates-recorded-description
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: busy-flag-or-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: describer-wiring-asserted-loosely
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: strand-stop-after-failed-state-write-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-module-line-names-only-tidy-agent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; there is no earlier ledger.
The review raised fourteen findings, F1 to F14, each carried above as an open entry.
F1 is the one BLOCKING finding, a stencil restating another producer's format.
F2 to F5 and F7 are MEDIUM design findings on the describe step's failure handling, its worktree-gone stop and its tick wiring test.
F6 is a MEDIUM consistency finding on the change description's Merge danger call.
F8 to F13 are LOW design findings, and F14 is a LOW consistency finding on the overview.
