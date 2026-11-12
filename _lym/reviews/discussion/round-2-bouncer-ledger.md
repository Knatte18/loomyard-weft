---
round: 2
ledger:
  - key: bridge-cursor-start-loses-first-message
    rounds: [1]
    status: resolved
  - key: lost-settle-past-hour-returns-delivered-mail
    rounds: [1]
    status: resolved
  - key: bridge-posts-wiped-from-outbox-record
    rounds: [1]
    status: resolved
  - key: bridge-channel-commands-run-on-both-machines
    rounds: [1]
    status: resolved
  - key: resync-adoption-trusts-any-envelope-poster
    rounds: [1]
    status: resolved
  - key: unmapped-channel-events-trigger-full-resyncs
    rounds: [1]
    status: resolved
  - key: first-contact-receiver-uninformed
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: acceptance-criteria-miss-stated-behaviours
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d4-holds-versus-d7-refusal-cap
    rounds: [1]
    status: resolved
  - key: once-per-pair-listing-false-on-hold-path
    rounds: [1]
    status: resolved
  - key: program-claim-bound-linux-only
    rounds: [1]
    status: resolved
  - key: from-naming-this-machine-held-forever
    rounds: [1]
    status: resolved
  - key: mail-to-retired-orch-through-recreated-bridge
    rounds: [1]
    status: resolved
  - key: app-id-lowercased-by-command-matching
    rounds: [1]
    status: resolved
  - key: machine-inbound-mail-in-creel-other-hub-branches
    rounds: [1]
    status: resolved
  - key: former-name-failing-checkname
    rounds: [1]
    status: resolved
  - key: operator-messages-rehandled-past-held-message
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: archived-bridge-channel-no-outcome
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: adopted-channel-start-read-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: returned-held-message-posted-after-hold-clears
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: machine-outbound-event-misread-by-existing-readers
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: relay-take-trigger-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: bridge-thread-replies-never-read
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: operator-change-not-invited-to-bridge-channels
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: resync-bridge-prefix-matches-orch-channels
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: refusal-count-home-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: button-off-criterion-needs-another-relay-orch
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: bridge-map-peer-app-duplicates-pin
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reply-sending-id-not-carried
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's BLOCKING cursor-start key and its other design keys no longer appear in round 2, so they are resolved.

Two round-1 keys reappear in round 2.
The first-contact notice now exists but lands only in the prime orch's channel, which may not exist (F6).
The acceptance criteria still miss two rate bounds (F7).

Round 2's new BLOCKING key comes from the fix to the cursor start.
Operator messages past a held peer message are reread on every poll, so answers repeat and a revoke can be undone (F1).
The related start-read bound (F3) and the archived-channel, returned-message and machine-outbound keys are new design gaps in the same read and post lifecycle.
The remaining keys are LOW additions and local corrections.
