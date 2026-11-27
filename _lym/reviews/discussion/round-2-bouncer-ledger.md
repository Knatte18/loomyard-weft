---
round: 2
ledger:
  - key: lock-wait-breaks-daemon-wakeups
    rounds: [1]
    status: resolved
  - key: parent-anchor-read-before-parent-placed
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: machine-start-opens-orch-windows-after-runs
    rounds: [1]
    status: resolved
  - key: bottom-row-under-panel
    rounds: [1]
    status: resolved
  - key: places-file-concurrent-writes-unspecified
    rounds: [1]
    status: resolved
  - key: overflow-overwrites-remembered-desktop
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: kwin-non-current-desktop-output-unpinned
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: konsole-pid-caption-separate-facts-deferred
    rounds: [1]
    status: resolved
  - key: roottile-probe-counted-as-prelaunch-check
    rounds: [1]
    status: resolved
  - key: window-waits-need-queries-outside-lock
    rounds: [1]
    status: resolved
  - key: per-user-lock-has-no-home
    rounds: [1]
    status: resolved
  - key: timeout-or-kill-leaves-script-and-half-built-desktop
    rounds: [1]
    status: resolved
  - key: window-command-lost-or-gained-with-placement
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: help-and-template-text-describe-in-verb-wait
    rounds: [1]
    status: resolved
  - key: renumber-collides-with-absent-order-keys
    rounds: [1]
    status: resolved
  - key: journal-since-granularity-claim-false
    rounds: [1]
    status: resolved
  - key: window-count-rule-differs-from-script
    rounds: [1]
    status: resolved
---

# Cross-round record

Round 1 raised every key in this ledger.
Round 2 raised no new key: each of its findings carries a round-1 key whose fix was incomplete.

The open work:

- The parent anchor: the anchor is read under the lock, but an unsettled parent entry's home or last desktop
  does not say where the parent lands.
- Overflow: the overflow keeps its home and order key, but a renumber of the landing desktop can rewrite the key.
- KWin facts: some are still listed as `Not yet observed`.
- `window_command`: the record does not say whether a SIGTERM exit runs it.

The round-2 review was incremental and lists further lens findings below BLOCKING that it did not raise.
They are in its `Valid findings not raised this round` section and have no ledger key.
