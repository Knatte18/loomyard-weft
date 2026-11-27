---
round: 3
ledger:
  - key: lock-wait-breaks-daemon-wakeups
    rounds: [1]
    status: resolved
  - key: parent-anchor-read-before-parent-placed
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: kwin-non-current-desktop-output-unpinned
    rounds: [1, 2]
    status: resolved
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
    status: resolved
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
  - key: konsole-resource-class-wrong-on-wayland
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: script-run-reaches-another-script
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: query-report-lacks-what-ordering-needs
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: renumber-rewrites-unsettled-window-keys
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

# Cross-round record

Round 1 raised its keys, and round 2 raised no new key.
Round 3 resolves every key open after round 2 and raises four new ones, all BLOCKING design findings.

The open work:

- The resource class D2 matches on: upstream source gives Konsole `org.kde.konsole` on Wayland, not `konsole`.
- Script identity: KWin reuses script ids, so overlapping runs can reach each other's scripts.
- The query report: it lacks the zone boxes, padding, window identity, current desktop and per-desktop tile membership.
- The renumber: it rewrites the order keys of reopened windows whose entries are unsettled.

The round-3 review lists further items below BLOCKING under `## Rejected`.
They have no ledger key.
