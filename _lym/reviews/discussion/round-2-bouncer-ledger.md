---
round: 2
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
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: poll-removal-strands-binary-reexec
    rounds: [1]
    status: resolved
  - key: upgrade-relay-exit-silent
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: setup-channel-flag-ways-forward
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reopen-no-entry-cursor
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: restart-cap-idle-lifetime
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-ensure-no-presence-baseline
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: live-hub-list-read-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: operator-invite-never-retried
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hint-atomic-write-races-drain
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: own-channel-bot-left-no-way-forward
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: thread-read-skips-parent-missing
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failed-reexec-leaves-socket-lock-wait
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: missing-token-notice-start-order
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: two-ensures-race-one-relay
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: beam-mail-routing-beams-read-fails
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-criterion-contradicts-rationale
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: slack-boundary-rule-only-in-notes
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: notes-bullet-restates-d3
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: support-log-open-risks-disagree
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 22 findings; round 2 raised 21, none BLOCKING.
Of the round-1 keys, four recur in round 2: F4 `upgrade-relay-exit-silent` (MEDIUM, the gating one),
F9 `not-in-channel-no-reinvite`, F10 `login-tokens-one-app-unchecked`, F15 `setup-channel-flag-ways-forward` (all LOW).
The other 18 round-1 keys are resolved.

Round 2 mapping of review IDs to new keys: F1 `hint-removal-not-before-read`, F2 `reopen-no-entry-cursor`,
F3 `restart-cap-idle-lifetime`, F5 `beam-ensure-no-presence-baseline`, F6 `live-hub-list-read-unbounded`,
F7 `operator-invite-never-retried`, F8 `hint-atomic-write-races-drain`, F11 `own-channel-bot-left-no-way-forward`,
F12 `thread-read-skips-parent-missing`, F13 `failed-reexec-leaves-socket-lock-wait`, F14 `missing-token-notice-start-order`,
F16 `two-ensures-race-one-relay`, F17 `beam-mail-routing-beams-read-fails`, F18 `d2-criterion-contradicts-rationale`,
F19 `slack-boundary-rule-only-in-notes`, F20 `notes-bullet-restates-d3`, F21 `support-log-open-risks-disagree`.
