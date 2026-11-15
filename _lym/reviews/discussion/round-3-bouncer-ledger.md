---
round: 3
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
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hold-entry-collides-with-operator-and-hibernate-pause
    rounds: [1]
    status: resolved
  - key: cancel-removes-record-holding-its-intent
    rounds: [1, 2, 3]
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
    status: resolved
  - key: d10-open-risk-defers-readable-fact
    rounds: [1]
    status: resolved
  - key: d7-bound-understates-pre-read-activity
    rounds: [1, 2]
    status: resolved
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
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-step4-cites-nonexistent-local-removal
    rounds: [2]
    status: resolved
  - key: upgrade-adopt-pause-writes-silent-hub-hold
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: mirror-interval-skips-last-turns
    rounds: [2]
    status: resolved
  - key: network-gate-holds-committed-transition-push
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: release-and-second-adopt-race-takeover-retry
    rounds: [2]
    status: resolved
  - key: adopter-branches-stay-behind-origin
    rounds: [2]
    status: resolved
  - key: d10-cross-machine-resume-unverified
    rounds: [2]
    status: resolved
  - key: orch-transcripts-to-public-weft
    rounds: [2, 3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: release-verb-against-operator-decision
    rounds: [2, 3]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: orch-up-during-hold-against-operator-decision
    rounds: [2]
    status: resolved
  - key: no-offline-progress-unconfirmed
    rounds: [2]
    status: resolved
  - key: d11-teardown-condition-reads-act-rule
    rounds: [2]
    status: resolved
  - key: refusals-without-way-forward
    rounds: [2]
    status: resolved
  - key: release-mid-adopt-leaves-no-dobby
    rounds: [2]
    status: resolved
  - key: q5-absences-disagree-with-d6
    rounds: [2]
    status: resolved
  - key: open-risks-restate-settled-calls
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 3 has 7 findings, down from 20.
The round 2 fixes for the hold, takeover, teardown, branch and mirror keys hold, and the mirror keys are moot now the parent drops the mirror and every cross-machine restore.

Three design keys keep recurring and make the loop circle:
the landing past the parent-side merge (the merge commit never reaches origin, so the adopter cannot finish the landing),
the shared pair push still sending the code side first for callers other than a transition,
and the cancel record that outlives a remove-cancel's pair.
Two LOW design keys from the network gate and the upgrade hold stay open.

Two Added-after-Discussion entries are still unfolded into the body:
the arbiter's removal of `--release` (needs a replacement takeover flag with stated bound and hold subjects)
and the parent's dropped orch transcript mirror (D10 reduces to the same-machine resume fix).
Open risks restate calls those entries settled and are dropped.
