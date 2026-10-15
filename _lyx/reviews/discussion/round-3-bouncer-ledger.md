---
round: 3
ledger:
  - key: batten-tamper-check-removal-unbounded
    rounds: [1, 2]
    status: resolved
  - key: loom-migrate-rewrite-permanent-reaches-run-settings
    rounds: [1]
    status: resolved
  - key: darn-tackler-settings-source-unnamed
    rounds: [1]
    status: resolved
  - key: batten-resume-halted-row-only-no-seam
    rounds: [1]
    status: resolved
  - key: d2-bound-line-overstates-hand-back
    rounds: [1]
    status: resolved
  - key: d3-no-go-seeded-run-premise-false
    rounds: [1, 2]
    status: resolved
  - key: acceptance-criteria-miss-d2-d7-behaviour
    rounds: [1]
    status: resolved
  - key: tackler-timeout-min-zero-meaning-open
    rounds: [1]
    status: resolved
  - key: abandoned-session-named-twice
    rounds: [1]
    status: resolved
  - key: d4-old-build-batten-adoption-mechanism
    rounds: [2]
    status: resolved
  - key: migration-leaves-stale-tackler-key-comments
    rounds: [2]
    status: resolved
  - key: migration-comments-stale-on-pinned-key-hubs
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised nine findings, round 2 four, round 3 one.
Every round 2 key is absent from round 3, so all are resolved.
The one open key is new in round 3: D5's migration rewrites only when all four tackler keys hold the old defaults, so a hub with any pinned key keeps comments that contradict inheriting `shed.yaml`.
Its fix splits the migration into a value rewrite and a per-key comment refresh, which changes the contract and needs fresh review.
