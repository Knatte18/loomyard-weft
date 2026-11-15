---
round: 1
ledger:
  - key: d4-handover-reads-conflicting-replay
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-hub-hold-entry-holds-what-act-rule-lets-act
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hold-without-live-adopter-has-no-release
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landing-past-merge-stranded-under-hold
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: hold-entry-collides-with-operator-and-hibernate-pause
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cancel-removes-record-holding-its-intent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: slug-pause-and-cancel-offline-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: owner-guard-unnamed-bolt-and-replayed-pause
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: other-detached-status-pushes-survive-d2
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: old-owner-mirror-outruns-adopter-restore
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-open-risk-defers-readable-fact
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-bound-understates-pre-read-activity
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-reread-ignores-other-subject-holds
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: upgrade-conversion-no-record-for-released-task
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pre-upgrade-orch-not-in-runproc
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: fix-try-limit-lives-in-cache
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: open-risk-restates-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: auto-mode-assumption-points-at-ledger
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 18 findings, none carried from an earlier round.
The three BLOCKING findings concern the takeover handover reading a conflicting replay, the hold's hub pause entry freezing the adopt-fix path, and a hold with no way out when its adopter is gone.
The MEDIUM design findings concern the pause record's shared slot, the cancel record's removal, offline slug verbs, the unnamed Bolt in the owner guard, detached status pushes, the old owner's mirror and a deferred fact.
The remaining findings are LOW or NIT wording and gap corrections.
