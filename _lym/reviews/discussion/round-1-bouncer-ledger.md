---
round: 1
ledger:
  - key: hint-dir-not-watched
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: missing-thread-cursor-replay
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: wake-on-own-writes-no-floor
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: thread-broadcast-forwarded-twice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hint-channel-unchecked-against-map
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: unreadable-live-map-outcome
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stalled-holder-lost-hint-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: channel-name-collision-across-machines
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: login-tokens-one-app-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: poll-removal-strands-binary-reexec
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: upgrade-relay-exit-silent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-one-check-misstates-start-path
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: way-forward-shortname-refuses
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stale-line-channel-undefined
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: not-in-channel-no-reinvite
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: setup-channel-flag-ways-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: holder-hub-paths-seam-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: login-prompt-task-landed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: missing-acceptance-bullets
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risks-line-no-risk
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: constraints-unneeded-slack-facts
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: d8-refusal-cap-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review: 22 findings, all open.
Keys map to review IDs in order: F1 `hint-dir-not-watched`, F2 `missing-thread-cursor-replay`, F3 `wake-on-own-writes-no-floor`,
F4 `thread-broadcast-forwarded-twice`, F5 `hint-channel-unchecked-against-map`, F6 `unreadable-live-map-outcome`,
F7 `stalled-holder-lost-hint-unbounded`, F8 `channel-name-collision-across-machines`, F9 `login-tokens-one-app-unchecked`,
F10 `poll-removal-strands-binary-reexec`, F11 `upgrade-relay-exit-silent`, F12 `d3-one-check-misstates-start-path`,
F13 `way-forward-shortname-refuses`, F14 `stale-line-channel-undefined`, F15 `not-in-channel-no-reinvite`,
F16 `setup-channel-flag-ways-forward`, F17 `holder-hub-paths-seam-unstated`, F18 `login-prompt-task-landed`,
F19 `missing-acceptance-bullets`, F20 `open-risks-line-no-risk`, F21 `constraints-unneeded-slack-facts`, F22 `d8-refusal-cap-unnamed`.
