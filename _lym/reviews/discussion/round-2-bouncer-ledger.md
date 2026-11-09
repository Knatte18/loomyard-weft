---
round: 2
ledger:
  - key: here-skips-unfinished-adopt-record
    rounds: [1]
    status: resolved
  - key: d4-uncovered-root-inheriting-beams
    rounds: [1]
    status: resolved
  - key: discard-races-starting-worker
    rounds: [1]
    status: resolved
  - key: stale-resume-takes-owner-from-third-machine
    rounds: [1]
    status: resolved
  - key: hidden-worker-verb-bound-false
    rounds: [1]
    status: resolved
  - key: d10-stale-bolt-bound-misstates-beam-start
    rounds: [1]
    status: resolved
  - key: d11-archive-tags-read-as-runs
    rounds: [1]
    status: resolved
  - key: d11-owner-machine-claim-uncited
    rounds: [1]
    status: resolved
  - key: anchor-dobby-end-kills-worker
    rounds: [1]
    status: resolved
  - key: stopped-record-exit-code-unstated
    rounds: [1]
    status: resolved
  - key: record-holds-no-envelope
    rounds: [1]
    status: resolved
  - key: losing-caller-follows-stale-done-record
    rounds: [1]
    status: resolved
  - key: record-write-wipes-resume-state
    rounds: [1]
    status: resolved
  - key: flag-merge-yields-both-local-flags
    rounds: [1]
    status: resolved
  - key: restart-at-reconcile-repauses-resumed-runs
    rounds: [1]
    status: resolved
  - key: d4-subject-test-undefined-for-all-record
    rounds: [1]
    status: resolved
  - key: d5-legacy-check-bound-understated
    rounds: [1]
    status: resolved
  - key: d6-no-run-claim-misses-front-tasks
    rounds: [1]
    status: resolved
  - key: failed-owner-pause-lets-pass-start
    rounds: [1]
    status: resolved
  - key: d11-bound-misplaces-grace-window
    rounds: [1]
    status: resolved
  - key: archives-lists-teardown-tags-as-old-owner
    rounds: [1]
    status: resolved
  - key: d12-names-no-repository-side
    rounds: [1]
    status: resolved
  - key: review-criterion-omits-dobby-orch-pair-rule
    rounds: [1]
    status: resolved
  - key: pause-before-archive-reason-duplicated
    rounds: [1]
    status: resolved
  - key: d11-old-owner-archives-nothing-without-dobby
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-adopted-beam-own-owner-pause-outlives-adopt
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-record-keeps-no-step-outcomes
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-criterion-drops-exceptions
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-discard-race-criterion-one-interleaving
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's 24 findings did not recur in round 2; the fresh review raised none of them, so all are carried as resolved.
Round 2 raised five new findings.
F1 is a MEDIUM design finding: D11 archives only through the old owner's running dobby, which never comes back for a machine that was off when the owner moved.
It is first raised in round 2, so Go carries it into Open risks rather than blocking.
F2 is a LOW design finding about an adopted beam's own `owner` pause entry that D5 never converts.
F3 to F5 are LOW consistency findings that restate the record's own text: missing step outcomes in D2, a D5 criterion that drops D5's exceptions, and a discard-race criterion that covers one interleaving.
