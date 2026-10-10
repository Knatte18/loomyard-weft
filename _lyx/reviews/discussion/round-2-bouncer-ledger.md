---
round: 2
ledger:
  - key: d2-detached-daemon-inherits-log-file
    rounds: [1]
    status: resolved
  - key: d4-zero-pin-rebuild-runs-in-attach-preflight
    rounds: [1]
    status: resolved
  - key: d7-dev-null-config-drop-unbounded
    rounds: [1]
    status: resolved
  - key: d3-tmux-agreement-premise-unchecked
    rounds: [1]
    status: resolved
  - key: d5-poll-mode-bound-overstates-transience
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d4-windows-unset-path-wording
    rounds: [1]
    status: resolved
  - key: d6-socket-limit-is-linux-not-posix
    rounds: [1]
    status: resolved
  - key: d7-attach-preflight-shell-resolution
    rounds: [1]
    status: resolved
  - key: d8-unmeasured-attribution-presented-as-fact
    rounds: [1]
    status: resolved
  - key: d6-d7-rejected-alternative-narrative
    rounds: [1]
    status: resolved
  - key: closes-495-and-lint-comments-not-plan-requirements
    rounds: [1]
    status: resolved
  - key: d6-socket-path-not-realpath
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-tmux-exit-status-misstated
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: d5-error-deferral-degraded-ticks-unclassified
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: d7-older-lyx-server-keeps-operator-config
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: d2-detached-debug-route-not-a-real-surface
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's eleven findings are no longer raised in round 2, apart from the D5 bound, which round 2's F4 re-raises on a narrower point (the hook-install case).
The other ten are resolved, including the round 1 BLOCKING on D2's inherited `LYX_LOG_FILE` and the two MEDIUMs on D4's pre-flight rebuild and D7's `-f /dev/null` bound.
Round 2 adds five keys: F1 (D6 socket path not realpath-resolved, design MEDIUM), F2 (D6 exit-status premise, design LOW), F3 (D5 error, deferral and degraded ticks, decision LOW), F5 (D7 older-lyx server keeps the operator's tmux config, scope LOW) and F6 (D2 detached Debug route, consistency LOW).
F1 is first raised in round 2, so Go carries it into the decision record's `## Open risks`.
