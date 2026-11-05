---
round: 2
ledger:
  - key: deliver-confirm-local-sender-resolution
    rounds: [1]
    status: resolved
  - key: overlapping-deliveries-stored-twice
    rounds: [1]
    status: resolved
  - key: returned-message-still-confirms-yes
    rounds: [1]
    status: resolved
  - key: creel-read-text-shows-no-stamp
    rounds: [1]
    status: resolved
  - key: uncommitted-config-edit-grants-trust
    rounds: [1]
    status: resolved
  - key: operator-text-unverified-mail-override
    rounds: [1]
    status: resolved
  - key: reply-awaiting-outbound-carried-forever
    rounds: [1]
    status: resolved
  - key: receiver-rules-before-duplicate-check
    rounds: [1]
    status: resolved
  - key: d1-prunes-live-hub-on-read-error
    rounds: [1]
    status: resolved
  - key: downgrade-loses-outbound-messages
    rounds: [1]
    status: resolved
  - key: confirm-tuple-omits-expects-reply
    rounds: [1]
    status: resolved
  - key: probe-flood-exposure-unstated
    rounds: [1]
    status: resolved
  - key: open-risk-sandbox-closed-area-settled
    rounds: [1]
    status: resolved
  - key: acceptance-gaps-d4-d6
    rounds: [1]
    status: resolved
  - key: d1-rationale-relitigates-rejected
    rounds: [1]
    status: resolved
  - key: d3-drop-lacks-journal-and-orch-notice
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: wait-reply-output-carries-no-trust
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stored-message-returned-undelivered
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: running-sessions-keep-old-trust-sentence
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: downgraded-receiver-shows-mail-unstamped
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: duplicate-key-assumes-ids-never-reused
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: cross-hub-reply-size-check-unstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: open-risks-line-already-settled
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: config-load-missing-from-cwd-constraint
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's fifteen findings were all addressed and none was raised again in round 2, so each is carried as resolved.
Round 2 raised nine new keys.
The blocking one is that D3 does not carry the operator's added drop rule: a journal entry and an orch notice, with a flood bound on the notice.
Four MEDIUM design findings concern the `--wait-reply` output lacking a trust stamp, a stored message later returned undelivered, running sessions keeping the old trust sentence, and a downgraded receiver showing cross-hub mail unstamped.
The rest are LOW: the duplicate key assuming ids are never reused, the unstated cross-hub reply size check, the Open risks line the operator already settled, and the config load missing from the cwd-resolution constraint.
