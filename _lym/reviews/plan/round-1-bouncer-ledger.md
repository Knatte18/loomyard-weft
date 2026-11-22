---
round: 1
ledger:
  - key: drift-passcommand-ended-ask-halt
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: shedrecipe-beamawait-fixture-seams
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card7-wrong-overview-sentence
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reaskparent-poll-no-backoff
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: prreview-stays-open-mail-to-stopped-session
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goto-orphans-live-pr-review-session
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: driftengine-doc-untrue
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card7-forcing-no-stated-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: goto-reset-before-move-certain
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: goto-comment-says-commits-nothing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: atorpastprreview-unreachable-branch
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: parent-answer-refusal-row-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: clearparentask-unreachable-refusals
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beamsetting-doc-untargeted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prreview-session-test-asserts-stencil-wording
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prcontinue-header-untargeted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: burlerengine-doc-cleared-ask
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 17 findings against the autonomy-mode plan.
Two are BLOCKING scope gaps: the diff-command drift judge keeps the old pass on an ended ask, and card 15 leaves the shedrecipe fixture without its new seams.
The design-class MEDIUM findings concern card 7's overview edit and unstated bound, the ask re-wait's polling, the stays-open mail's target session, and a goto leaving a live session behind.
The remaining findings are LOW or NIT consistency and test-economy corrections.
