---
round: 1
ledger:
  - key: d7-relaunch-no-mechanism
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pinned-no-entry-blocked-pin
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hub-pin-switched-on-limit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-copied-transcript-resume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-vs-retryapierror
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: orch-watcher-cannot-restart
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-prune-no-input
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: blocked-until-vs-windows-eligibility
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-forged-input-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: openlaunch-no-status-line
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-openlaunch-copy-sentence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-reset-zero-weight
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-switch-partial-counter
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: retired-account-no-move
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: config-dir-links-unresolved
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d7-set-bound-reach
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beams-pin-open-risk-writer-choice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: rationales-reargue-rejected
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d10-landing-notes-criterion
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised nineteen findings, mapped one to one onto the keys above (F1 to F19 in review order).
The BLOCKING one is D7's relaunch, which has no mechanism in shuttle and needs either a stop-verb mode or a declared PATTERN change.
Most MEDIUM findings are design gaps in D2 to D7 and D10: pin behaviour, the copied-transcript fallback, ordering against `retryAPIError`, the orch watcher, the prune input and `OpenLaunch` settings.
The LOW findings are local corrections whose text the review already fixes.
No key recurs, since this is the first round.
