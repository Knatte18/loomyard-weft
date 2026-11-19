---
round: 2
focus:
  - "internal/shedadapters/parentseat.go and burlerreview.go: settle whether the Code-Review parent ask goes out after the round's own review and fixer report exist and before that round's Bouncer judges, naming those same files, and whether the parent's findings reach the Bouncer's judgement."
  - "internal/driftengine/judgeparent.go agreementGate and agreement: settle whether a satisfied or findings answer passes only while judgement.yaml still holds the rulings the ask presented, including after a judge respawn under the same spawn id."
  - "internal/driftengine/verify.go parentAgreement: settle whether the parent wait is capped before the judge's own timeout, so a silent parent leaves the rulings as written instead of a Stuck."
  - "internal/shedadapters/parentseat.go parentAskDue: settle whether a Discussion-Review seat judges against the latest ask of its own kind when the shared ask record's latest ask belongs to Code-Review or the drift judge."
  - "Tests added for the seats, the table append and the gate rows: settle whether each pins behavior no other test covers, and whether tests the redundancy report lists as contained carry a keep marker only where they pin what their cover cannot see."
---

Focus targets the fixes to round 1's gating findings and the places they touch.
