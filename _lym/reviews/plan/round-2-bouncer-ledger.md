---
round: 2
ledger:
  - key: drift-passcommand-ended-ask-halt
    rounds: [1]
    status: resolved
  - key: shedrecipe-beamawait-fixture-seams
    rounds: [1]
    status: resolved
  - key: card7-wrong-overview-sentence
    rounds: [1]
    status: resolved
  - key: reaskparent-poll-no-backoff
    rounds: [1]
    status: resolved
  - key: prreview-stays-open-mail-to-stopped-session
    rounds: [1]
    status: resolved
  - key: goto-orphans-live-pr-review-session
    rounds: [1]
    status: resolved
  - key: driftengine-doc-untrue
    rounds: [1]
    status: resolved
  - key: card7-forcing-no-stated-bound
    rounds: [1]
    status: resolved
  - key: goto-reset-before-move-certain
    rounds: [1]
    status: resolved
  - key: goto-comment-says-commits-nothing
    rounds: [1]
    status: resolved
  - key: atorpastprreview-unreachable-branch
    rounds: [1]
    status: resolved
  - key: parent-answer-refusal-row-unreachable
    rounds: [1]
    status: resolved
  - key: clearparentask-unreachable-refusals
    rounds: [1]
    status: resolved
  - key: beamsetting-doc-untargeted
    rounds: [1]
    status: resolved
  - key: prreview-session-test-asserts-stencil-wording
    rounds: [1]
    status: resolved
  - key: prcontinue-header-untargeted
    rounds: [1]
    status: resolved
  - key: burlerengine-doc-cleared-ask
    rounds: [1]
    status: resolved
  - key: goto-stop-reads-unwired-reed-runner
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised 17 findings; the round-2 review marks every one resolved.
Round 2 raises one new BLOCKING scope finding: card 14's `prReviewGotoAdmitted` builds its session stop from `c.reed` and `c.runner`,
which the `goto` verb's lightweight wiring leaves nil.
The review's fix routes the stop through `stepStopRunner` resolved at call time, a local correction.
