---
round: 2
ledger:
  - key: shell-sigkill-reads-as-memory-cap-kill
    rounds: [1]
    status: resolved
  - key: guard-kill-in-runalone-reads-as-cap-kill
    rounds: [1]
    status: resolved
  - key: redundancy-report-step-unguarded
    rounds: [1]
    status: resolved
  - key: tier-run-never-names-killed-test
    rounds: [1]
    status: resolved
  - key: unsettable-memory-cap-no-cause-or-way-forward
    rounds: [1]
    status: resolved
  - key: tier-run-memory-kill-wiring-untested
    rounds: [1]
    status: resolved
  - key: tiers-still-uncapped-no-default
    rounds: [1]
    status: resolved
  - key: guardowntests-start-stop-untested
    rounds: [1]
    status: resolved
  - key: cpu-watchdog-sentence-misplaced-after-memory-section
    rounds: [1]
    status: resolved
  - key: stderr-field-comment-misses-new-lines
    rounds: [1]
    status: resolved
  - key: oom-poll-wakes-every-20ms
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: scope-start-read-deadline-misreads-slow-start
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: guard-kill-of-inner-binary-leaves-outer-pid-unnamed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: oom-kill-count-includes-kills-outside-the-scope-limit
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: change-description-contradicts-diff
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: testmap-guard-kill-documented-as-not-judged
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: memory-mb-docs-miss-data-segment-fallback
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's ten keys are not raised again by the round-2 review and are carried as resolved.
Round 2 raised seven new keys.
The BLOCKING one is the 20 ms `memory.events` poll in `runMemoryCapped`, which breaks the one-second wake floor of `PATTERN-daemon-wakeups`; its fix picks between an fswatch wake and a slower poll, so it is a design change.
Two MEDIUM design findings concern the scope-start read deadline and the outer pid of a guard-killed inner test binary.
The remaining four are an OOM-kill attribution LOW, a MEDIUM change-description contradiction, and two LOW doc findings.
