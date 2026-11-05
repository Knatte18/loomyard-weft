---
round: 3
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
    status: resolved
  - key: wait-reply-output-carries-no-trust
    rounds: [2]
    status: resolved
  - key: stored-message-returned-undelivered
    rounds: [2]
    status: resolved
  - key: running-sessions-keep-old-trust-sentence
    rounds: [2, 3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: downgraded-receiver-shows-mail-unstamped
    rounds: [2]
    status: resolved
  - key: duplicate-key-assumes-ids-never-reused
    rounds: [2]
    status: resolved
  - key: cross-hub-reply-size-check-unstated
    rounds: [2]
    status: resolved
  - key: open-risks-line-already-settled
    rounds: [2]
    status: resolved
  - key: config-load-missing-from-cwd-constraint
    rounds: [2]
    status: resolved
  - key: daemon-start-retry-capped-by-time-only
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: drop-notice-bound-keyed-on-forged-strand
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: identical-message-after-id-restart-stored-nowhere
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: return-notice-relays-unbounded-reason-text
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1's fifteen findings and round 2's other keys were not raised again, so each is carried as resolved.
Round 2's `d3-drop-lacks-journal-and-orch-notice` and `duplicate-key-assumes-ids-never-reused` are resolved because round 3 confirms their fixes landed; round 3 raised narrower flaws next to them, filed as new keys.
Round 3 raised four new keys.
The BLOCKING one is that D2's worker restarts a receiving hub's daemon for an hour with no cap on the count of starts.
Two MEDIUM design findings concern the drop-notice bound keyed on a strand name the forger chooses, and an identical message after a sending hub's ids restart answered accepted and stored nowhere.
The LOW findings are the unbounded reason text in the return notice and the pre-deploy session bound, which still rests on a prediction about how a session reads a contradicting stamp.
