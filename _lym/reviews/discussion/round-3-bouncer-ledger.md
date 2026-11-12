---
round: 3
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
    status: resolved
  - key: acceptance-criteria-miss-stated-behaviours
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: archived-bridge-channel-no-outcome
    rounds: [2]
    status: resolved
  - key: adopted-channel-start-read-unbounded
    rounds: [2]
    status: resolved
  - key: returned-held-message-posted-after-hold-clears
    rounds: [2]
    status: resolved
  - key: machine-outbound-event-misread-by-existing-readers
    rounds: [2]
    status: resolved
  - key: relay-take-trigger-unstated
    rounds: [2]
    status: resolved
  - key: bridge-thread-replies-never-read
    rounds: [2]
    status: resolved
  - key: operator-change-not-invited-to-bridge-channels
    rounds: [2]
    status: resolved
  - key: resync-bridge-prefix-matches-orch-channels
    rounds: [2]
    status: resolved
  - key: refusal-count-home-unstated
    rounds: [2]
    status: resolved
  - key: button-off-criterion-needs-another-relay-orch
    rounds: [2]
    status: resolved
  - key: bridge-map-peer-app-duplicates-pin
    rounds: [2]
    status: resolved
  - key: reply-sending-id-not-carried
    rounds: [2]
    status: resolved
  - key: machine-outbound-hour-has-no-owner
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: bridge-archived-way-forward-impossible
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: relay-verb-journal-form-unstated
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bridge-channel-hold-state-has-no-home
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Rounds 1 and 2 reduced the finding count from 16 to 15, and round 3 to 4.
Every key from rounds 1 and 2 is absent from round 3, so all are resolved.

Round 3 raises four new design keys, all in the machine-outbound lifecycle and the bridge-channel hold.
The BLOCKING one is that D3 leaves no creel timer to return a machine-outbound message at its hour while the relay is down.
The MEDIUM keys are an impossible Slack way forward for an archived channel, an unstated journal form for the report and settle verbs, and a missing home for per-channel notice marks together with the delivery cursor after an approval.
