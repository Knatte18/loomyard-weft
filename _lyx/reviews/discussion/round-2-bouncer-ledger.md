---
round: 2
ledger:
  - key: confirmation-states-unknown-passes
    rounds: [1]
    status: resolved
  - key: no-live-pause-when-unowned
    rounds: [1]
    status: resolved
  - key: sleep-deny-wrong-verbs
    rounds: [1]
    status: resolved
  - key: sleep-deny-bound-missing
    rounds: [1]
    status: resolved
  - key: watch-verb-unguarded
    rounds: [1]
    status: resolved
  - key: daemon-liveness-test-unnamed
    rounds: [1]
    status: resolved
  - key: pause-grace-unstated
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: deliberation-in-decision-record
    rounds: [1]
    status: resolved
  - key: lift-without-resume-lines
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: confirmation-failure-lift-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two BLOCKING design findings on the pre-hibernate safety conditions, three MEDIUM design findings on the sleep-command deny and the `watch` verb, and four LOW corrections.
Round 2 did not re-raise any of those, and rejected the focus questions on the confirmation state set, the caller label, `watch` and liveness as non-defects, so they are resolved.
Round 2 raised one new MEDIUM design finding, that the wake and cancel lifts do not say the interrupted sessions are given their resume lines.
The remaining findings are LOW: Decision 6 omits the lift and notice on a confirmation failure, and two acceptance-criteria gaps (the sleep-deny bound, and the halted-before-Land and different-live-entry paths) continue the round-1 criteria gaps key.
