---
round: 1
ledger:
  - key: lock-wait-breaks-daemon-wakeups
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-anchor-read-before-parent-placed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: machine-start-opens-orch-windows-after-runs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bottom-row-under-panel
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: places-file-concurrent-writes-unspecified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overflow-overwrites-remembered-desktop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: kwin-non-current-desktop-output-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: konsole-pid-caption-separate-facts-deferred
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: roottile-probe-counted-as-prelaunch-check
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: window-waits-need-queries-outside-lock
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: per-user-lock-has-no-home
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: timeout-or-kill-leaves-script-and-half-built-desktop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: window-command-lost-or-gained-with-placement
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: help-and-template-text-describe-in-verb-wait
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: renumber-collides-with-absent-order-keys
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: journal-since-granularity-claim-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: window-count-rule-differs-from-script
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
---

# Cross-round record

Round 1 is the first round, so every key is new and open.

The open work groups into four areas:

- Ordering and waiting: the lock wait, the parent anchor, `lym machine start`'s window order,
  the signal each wait reads, and the lock's location.
- State and failure paths: the places file's writers, overflow, order keys, a timed-out or killed placement,
  and when `window_command` runs.
- Facts the design rests on: KWin behaviour off the current desktop and output, and Konsole's pid and caption.
- Geometry and text: the panel case, the window count rule, the probe's place among D1's checks,
  and stale help, template and rationale text.
