---
round: 1
ledger:
  - key: unwatched-timer-warn-every-wake
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: closed-watch-channel-fallback-untested
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-tmux-3-4-warns-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all new.
The BLOCKING one is the per-wake Warn in the watchdog's unwatched timer arm, which `logger.Episode` fixes locally.
The two LOW ones are a missing test row for the closed-channel path and a stale comment in the naming integration test.
