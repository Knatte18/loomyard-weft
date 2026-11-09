---
round: 1
ledger:
  - key: here-skips-unfinished-adopt-record
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-uncovered-root-inheriting-beams
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: discard-races-starting-worker
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-resume-takes-owner-from-third-machine
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hidden-worker-verb-bound-false
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-stale-bolt-bound-misstates-beam-start
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d11-archive-tags-read-as-runs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d11-owner-machine-claim-uncited
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: anchor-dobby-end-kills-worker
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: stopped-record-exit-code-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: record-holds-no-envelope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: losing-caller-follows-stale-done-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: record-write-wipes-resume-state
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: flag-merge-yields-both-local-flags
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: restart-at-reconcile-repauses-resumed-runs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-subject-test-undefined-for-all-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-legacy-check-bound-understated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d6-no-run-claim-misses-front-tasks
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: failed-owner-pause-lets-pass-start
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d11-bound-misplaces-grace-window
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: archives-lists-teardown-tags-as-old-owner
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d12-names-no-repository-side
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: review-criterion-omits-dobby-orch-pair-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pause-before-archive-reason-duplicated
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 24 findings, all still open.
The gating ones are two BLOCKING design findings, F1 (`--here` ends `owned_here` without reading an unfinished record) and F2 (D4 does not cover beams that inherit the prime's owner), and five MEDIUM design findings, F3 to F7.
F8 is a MEDIUM consistency finding that needs a rule-text citation or a PATTERN change bullet.
The remaining findings are LOW or NIT, mostly stated bounds and cross-references that do not match the code or another decision.
Several of the design fixes touch D1, D2 and D4, so the round-2 reviewers need to see those decisions again as a whole.
