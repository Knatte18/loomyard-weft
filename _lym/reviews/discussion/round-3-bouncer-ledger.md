---
round: 3
ledger:
  - key: relay-in-starter-pane-tree
    rounds: [1]
    status: resolved
  - key: cursorless-first-start-loses-messages
    rounds: [1]
    status: resolved
  - key: anchor-change-ends-relay
    rounds: [1, 2]
    status: resolved
  - key: slack-stop-undone-by-dobby
    rounds: [1]
    status: resolved
  - key: retention-bound-false
    rounds: [1]
    status: resolved
  - key: release-and-repost-never-drains
    rounds: [1, 2]
    status: resolved
  - key: reminder-wait-start-ambiguous
    rounds: [1, 2]
    status: resolved
  - key: presence-verbs-unbounded
    rounds: [1, 2]
    status: resolved
  - key: rate-limit-four-hubs
    rounds: [1]
    status: resolved
  - key: hold-delay-not-bounded-by-hour
    rounds: [1]
    status: resolved
  - key: slug-mail-to-torn-down-beam-lost
    rounds: [1]
    status: resolved
  - key: beam-dobby-reminder-no-answer-hint
    rounds: [1]
    status: resolved
  - key: slack-facts-unsourced
    rounds: [1]
    status: resolved
  - key: outbound-rationale-and-d6-criterion-overstate
    rounds: [1]
    status: resolved
  - key: dobby-pass-wrong-cadence
    rounds: [1]
    status: resolved
  - key: open-risk-restates-d4-bound
    rounds: [1]
    status: resolved
  - key: double-fork-departs-reap-children
    rounds: [2]
    status: resolved
  - key: failed-creel-post-undefined
    rounds: [2]
    status: resolved
  - key: external-mail-passes-sender-filter
    rounds: [2]
    status: resolved
  - key: cursor-not-keyed-by-channel
    rounds: [2]
    status: resolved
  - key: reexec-drops-single-instance-lock
    rounds: [2]
    status: resolved
  - key: channel-operator-id-repointable-via-config
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pane-check-unknown-pane-unspecified
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hidden-relay-verb-unbounded
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised sixteen findings, eight of them gating design MEDIUMs; round 2 narrowed to nine with one BLOCKING; round 3 narrows to three, all design.

Every round-2 key is resolved: the round-3 review raises none of them again, and its Rejected section settles the double fork, the failed creel post and the EXTERNAL sender filter.
The round-2 gap in the operator verbs' bounds is closed; round 3 finds two new gaps in the same area instead.

Three keys open, all first raised in round 3.
F1 is BLOCKING: the channel and operator ids sit in machine-layer config, which `lym config slack --layer machine` can write from the prime orch's pane, so D2's no-repoint bound is false.
F2 is MEDIUM: the pane-ancestry check does not say what it does while a live strand's pane process is unknown.
F3 is MEDIUM: the hidden `lym slack relay` verb has no audience, and the relay's reaction to a refused first creel read is unstated.
All three change a decision or contract, so each is `design-wrong` and its fix needs a fresh review.
