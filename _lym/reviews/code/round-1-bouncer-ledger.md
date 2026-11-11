---
round: 1
ledger:
  - key: adopt-beam-without-owner-not-covered
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: archive-tip-decision-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: owner-archive-pairs-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: adopt-hold-skipped-steps-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: archive-failure-episode-never-resets
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: watch-orch-hold-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: bolt-pull-failure-run-pause-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: archive-lock-multipair-nopush-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 maps F1 to `adopt-beam-without-owner-not-covered`, F2 to `archive-tip-decision-rows-missing`,
F3 to `owner-archive-pairs-untested`, F4 to `adopt-hold-skipped-steps-unpinned`,
F5 to `archive-failure-episode-never-resets`, F6 to `watch-orch-hold-wiring-untested`,
F7 to `bolt-pull-failure-run-pause-untested`, F8 to `archive-lock-multipair-nopush-untested`.
F1 is a logic error in subject resolution, so its fix changes a contract and needs fresh review.
The rest are local corrections, mostly missing test rows, that the finding text already fixes.
