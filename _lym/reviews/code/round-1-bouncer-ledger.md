---
round: 1
ledger:
  - key: starttestmap-endprocess-unregistered-child
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: testmapstep-error-aborts-tick
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: beamdown-skips-refresh-end-when-dobby-dead
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: starttestmap-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: move-skip-of-testmap-step-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: anchor-move-effect-on-refresh-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refresh-count-grows-while-every-zero
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prime-only-wiring-and-tidy-pause-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testmap-error-paths-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 maps F1 to F9 in order onto the keys above.
F1 is the only BLOCKING finding.
F2, F3 and F4 are the other gating findings.
F3 is marked `design-wrong` because its fix adds a new stop step and a new `orchCLI` seam.
