---
round: 2
ledger:
  - key: batten-tamper-check-removal-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
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
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: migration-leaves-stale-tackler-key-comments
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings; round 2 no longer raises seven of them, so they are resolved.
Two keys persist.
The D4 tamper-removal finding returns as BLOCKING because the named bound, shuttle's autonomous-posture denies, covers no file writes, `git add` or `git commit`.
The D3 premise finding returns as MEDIUM because drive reports already exist in llm-seeded runs, so only the Go-driver-only records change home.
Two LOW findings are new: D4's old-build adoption mechanism is mis-stated, and the loom.yaml migration leaves three stale key comments.
