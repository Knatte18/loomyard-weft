---
round: 1
ledger:
  - key: relay-in-starter-pane-tree
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cursorless-first-start-loses-messages
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: anchor-change-ends-relay
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: slack-stop-undone-by-dobby
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retention-bound-false
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: release-and-repost-never-drains
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reminder-wait-start-ambiguous
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: presence-verbs-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: rate-limit-four-hubs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hold-delay-not-bounded-by-hour
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: slug-mail-to-torn-down-beam-lost
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beam-dobby-reminder-no-answer-hint
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: slack-facts-unsourced
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: outbound-rationale-and-d6-criterion-overstate
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dobby-pass-wrong-cadence
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risk-restates-d4-bound
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 was the first round; no earlier ledger exists.
Eight design-class MEDIUM findings gate: the relay's process-tree premise for its SLACK claim, the cursorless first start, the anchor change ending the relay, `stop` undone by the dobby, the false retention bound, release-and-repost never draining, the ambiguous reminder wait start, and the unbounded presence verbs.
Eight LOW findings are local corrections to bounds, sources and wording.
