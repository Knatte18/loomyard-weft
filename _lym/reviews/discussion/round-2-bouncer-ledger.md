---
round: 2
ledger:
  - key: d4-handover-reads-conflicting-replay
    rounds: [1]
    status: resolved
  - key: d7-hub-hold-entry-holds-what-act-rule-lets-act
    rounds: [1]
    status: resolved
  - key: hold-without-live-adopter-has-no-release
    rounds: [1]
    status: resolved
  - key: landing-past-merge-stranded-under-hold
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hold-entry-collides-with-operator-and-hibernate-pause
    rounds: [1]
    status: resolved
  - key: cancel-removes-record-holding-its-intent
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: slug-pause-and-cancel-offline-unstated
    rounds: [1]
    status: resolved
  - key: owner-guard-unnamed-bolt-and-replayed-pause
    rounds: [1]
    status: resolved
  - key: other-detached-status-pushes-survive-d2
    rounds: [1]
    status: resolved
  - key: old-owner-mirror-outruns-adopter-restore
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-open-risk-defers-readable-fact
    rounds: [1]
    status: resolved
  - key: d7-bound-understates-pre-read-activity
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d8-reread-ignores-other-subject-holds
    rounds: [1]
    status: resolved
  - key: upgrade-conversion-no-record-for-released-task
    rounds: [1]
    status: resolved
  - key: pre-upgrade-orch-not-in-runproc
    rounds: [1]
    status: resolved
  - key: fix-try-limit-lives-in-cache
    rounds: [1]
    status: resolved
  - key: open-risk-restates-scope
    rounds: [1]
    status: resolved
  - key: auto-mode-assumption-points-at-ledger
    rounds: [1]
    status: resolved
  - key: pair-push-code-side-before-records-side
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-step4-cites-nonexistent-local-removal
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: upgrade-adopt-pause-writes-silent-hub-hold
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: mirror-interval-skips-last-turns
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: network-gate-holds-committed-transition-push
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: release-and-second-adopt-race-takeover-retry
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopter-branches-stay-behind-origin
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-cross-machine-resume-unverified
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: orch-transcripts-to-public-weft
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: release-verb-against-operator-decision
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: orch-up-during-hold-against-operator-decision
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: no-offline-progress-unconfirmed
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: d11-teardown-condition-reads-act-rule
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refusals-without-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: release-mid-adopt-leaves-no-dobby
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: q5-absences-disagree-with-d6
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's findings were fixed; of its 18 keys, four reappear in round 2 (landing past the merge, the cancel record, the old owner's mirror, the hold lag bound), the rest are resolved.
The landing finding is now BLOCKING design: after a takeover the carve-out holds on neither machine.
Round 2 added a second BLOCKING design finding, the pair push sending the code side before the records side.
Eleven further MEDIUM or LOW design keys are new, including the upgrade's silent hub hold, the missing trailing mirror, the network gate on a transition's push, the release race and the adopter's stale local branches.
Four `decision` keys (public weft transcripts, `--release`, the orch up during a hold, offline progress) go to the run's parent.
Four consistency keys are mechanical wording corrections.
