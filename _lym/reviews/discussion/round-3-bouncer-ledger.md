---
round: 3
ledger:
  - key: hint-dir-not-watched
    rounds: [1]
    status: resolved
  - key: missing-thread-cursor-replay
    rounds: [1]
    status: resolved
  - key: wake-on-own-writes-no-floor
    rounds: [1]
    status: resolved
  - key: thread-broadcast-forwarded-twice
    rounds: [1]
    status: resolved
  - key: hint-channel-unchecked-against-map
    rounds: [1]
    status: resolved
  - key: unreadable-live-map-outcome
    rounds: [1]
    status: resolved
  - key: stalled-holder-lost-hint-unbounded
    rounds: [1]
    status: resolved
  - key: channel-name-collision-across-machines
    rounds: [1]
    status: resolved
  - key: login-tokens-one-app-unchecked
    rounds: [1, 2]
    status: resolved
  - key: poll-removal-strands-binary-reexec
    rounds: [1]
    status: resolved
  - key: upgrade-relay-exit-silent
    rounds: [1, 2]
    status: resolved
  - key: d3-one-check-misstates-start-path
    rounds: [1]
    status: resolved
  - key: way-forward-shortname-refuses
    rounds: [1]
    status: resolved
  - key: stale-line-channel-undefined
    rounds: [1]
    status: resolved
  - key: not-in-channel-no-reinvite
    rounds: [1, 2]
    status: resolved
  - key: setup-channel-flag-ways-forward
    rounds: [1, 2]
    status: resolved
  - key: holder-hub-paths-seam-unstated
    rounds: [1]
    status: resolved
  - key: login-prompt-task-landed
    rounds: [1]
    status: resolved
  - key: missing-acceptance-bullets
    rounds: [1]
    status: resolved
  - key: open-risks-line-no-risk
    rounds: [1]
    status: resolved
  - key: constraints-unneeded-slack-facts
    rounds: [1]
    status: resolved
  - key: d8-refusal-cap-unnamed
    rounds: [1]
    status: resolved
  - key: hint-removal-not-before-read
    rounds: [2]
    status: resolved
  - key: reopen-no-entry-cursor
    rounds: [2]
    status: resolved
  - key: restart-cap-idle-lifetime
    rounds: [2]
    status: resolved
  - key: beam-ensure-no-presence-baseline
    rounds: [2]
    status: resolved
  - key: live-hub-list-read-unbounded
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: operator-invite-never-retried
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hint-atomic-write-races-drain
    rounds: [2]
    status: resolved
  - key: own-channel-bot-left-no-way-forward
    rounds: [2]
    status: resolved
  - key: thread-read-skips-parent-missing
    rounds: [2]
    status: resolved
  - key: failed-reexec-leaves-socket-lock-wait
    rounds: [2]
    status: resolved
  - key: missing-token-notice-start-order
    rounds: [2]
    status: resolved
  - key: two-ensures-race-one-relay
    rounds: [2]
    status: resolved
  - key: beam-mail-routing-beams-read-fails
    rounds: [2]
    status: resolved
  - key: d2-criterion-contradicts-rationale
    rounds: [2]
    status: resolved
  - key: slack-boundary-rule-only-in-notes
    rounds: [2]
    status: resolved
  - key: notes-bullet-restates-d3
    rounds: [2]
    status: resolved
  - key: support-log-open-risks-disagree
    rounds: [2]
    status: resolved
  - key: failed-hinted-read-never-retried
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 3 raised three findings, all `design`, after 22 in round 1 and 21 in round 2.
Every round-2 key the fresh review did not raise again is resolved.

Round 3 mapping of review IDs to keys:
F1 `failed-hinted-read-never-retried` (new, BLOCKING; the hint removal before the read is correct, the gap is the retry after a failed read),
F2 `operator-invite-never-retried` (reopened from round 2; no event re-ensures a held channel),
F3 `live-hub-list-read-unbounded` (reopened from round 2; a failing `hubregistry.Live` read gives the holder no notice).
