---
round: 1
ledger:
  - key: removestrand-cascade-flag-unpinned
    rounds: [1]
    status: resolved
  - key: sidecar-delete-yagni
    rounds: [1]
    status: resolved
  - key: invariant-amendment-silent-on-behaviour-subbullet
    rounds: [1]
    status: resolved
  - key: goal-overstates-never-exactly-one
    rounds: [1]
    status: resolved
  - key: step-driver-read-criterion-untestable
    rounds: [1]
    status: resolved
  - key: failed-reed-up-criterion-missing
    rounds: [1]
    status: resolved
  - key: helper-boundary-decision-is-plan-detail
    rounds: [1]
    status: resolved
---

Round 1 produced seven findings.
The MEDIUM one: the removal's recursive flag was unpinned, so a plan writer could have picked the cascading form.
The rest were LOW or NIT: a YAGNI sidecar delete, an invariant amendment silent on its behaviour sub-bullet, an overstated Goal, an untestable criterion, a missing failed-`Up` criterion, and a Decision that was plan detail.
The fixer resolved all seven in the same round.
Two of them, the sidecar delete and the helper-boundary Decision, were resolved by removal or relocation rather than addition.
No finding is open going into round 2.
