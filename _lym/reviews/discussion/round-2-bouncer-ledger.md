---
round: 2
ledger:
  - key: relay-in-starter-pane-tree
    rounds: [1]
    status: resolved
  - key: cursorless-first-start-loses-messages
    rounds: [1]
    status: resolved
  - key: anchor-change-ends-relay
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: slack-stop-undone-by-dobby
    rounds: [1]
    status: resolved
  - key: retention-bound-false
    rounds: [1]
    status: resolved
  - key: release-and-repost-never-drains
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reminder-wait-start-ambiguous
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: presence-verbs-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: failed-creel-post-undefined
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: external-mail-passes-sender-filter
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cursor-not-keyed-by-channel
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: reexec-drops-single-instance-lock
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised sixteen findings, eight of them gating design MEDIUMs.
Round 2's review shows the process-tree premise, the cursorless first start, the stop-versus-dobby restart and the retention bound holding; those keys are resolved, as are the LOW wording and source keys it does not raise again.

Four round-1 keys continue under new defects.
`release-and-repost-never-drains` continues as F2, the posted-id record against creel's acknowledgement window.
`presence-verbs-unbounded` continues as F4, with `login`, `setup` and `start` still unbounded.
`anchor-change-ends-relay` continues as F6, a consistency finding about the restart paths the criterion names.
`reminder-wait-start-ambiguous` continues as F7, a pause and resume that dobby does not observe.

Five keys are first raised in round 2: F1 the double fork against `PATTERN-reap-children`, F3 a failed creel post, F5 EXTERNAL mail passing the sender filter, F8 the cursor not keyed by channel, F9 the lock dropped across the re-exec.
F1 is BLOCKING, so the loop continues.
