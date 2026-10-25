---
round: 2
ledger:
  - key: d8-escalation-bypasses-judgment-refusal
    rounds: [1, 2]
    status: resolved
  - key: d8-settled-generation-rule-has-no-exit
    rounds: [1]
    status: resolved
  - key: d6-lens-reads-exceed-six-call-cap
    rounds: [1]
    status: resolved
  - key: d11-merge-in-fixer-attempt-record-lost
    rounds: [1]
    status: resolved
  - key: d1-landing-approval-is-parents-call
    rounds: [1]
    status: resolved
  - key: d8-hard-cap-reescalation-strips-decision-cause
    rounds: [1]
    status: resolved
  - key: d8-fix-row-silence-switch-unstated
    rounds: [1]
    status: resolved
  - key: d10-writer-wait-not-woken-and-silent-stop
    rounds: [1]
    status: resolved
  - key: d5-other-fans-and-hub-lens-layers
    rounds: [1]
    status: resolved
  - key: d8-accept-carries-answered-findings
    rounds: [1]
    status: resolved
  - key: d6-landed-commit-routes-two-ways
    rounds: [1]
    status: resolved
  - key: d8-d7-bound-is-prompt-text-only
    rounds: [1]
    status: resolved
  - key: d8-accept-exemption-misses-ledger-blocking-check
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d8-continue-bypasses-dropped-key-guard
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-refused-accept-forgets-parent-answer
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-next-fixer-carrier-does-not-exist
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-added-after-discussion-snapshot-lost-on-retire
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-no-parent-stop-defers-gate-behaviour
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d10-timeout-bound-names-nonexistent-respawn
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-refused-accept-not-a-continue-on-disk
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 2 settled most of round 1: the focus verdicts found the settled-generation re-entry, the six-call lens cap, the Merge-In attempt record and the hard-cap re-escalation sound, and the round-1 LOW keys were not re-raised.
The round-1 BLOCKING key on D8's escalation versus the judgment refusal is resolved in its original form, but its repair opened a family of new D8 findings.
The BLOCKING one: the exemption for decision findings covers the review check but not the ledger's open-BLOCKING entry check, so a parent cannot pass a BLOCKING decision finding.
Four MEDIUM D8 findings follow from the same area: a continue bypasses the dropped-key guard, a refused accept forgets the parent's answer, the fixer carrier named for refused findings does not exist, and the Added-after-Discussion snapshot is lost when the escalation record is retired.
D10's no-parent stop defers Darn-Write's gate behaviour to Plan (MEDIUM), and its timeout bound names a respawn that does not happen (LOW).
