---
round: 1
ledger:
  - key: d2-detached-daemon-inherits-log-file
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d4-zero-pin-rebuild-runs-in-attach-preflight
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-dev-null-config-drop-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-tmux-agreement-premise-unchecked
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d5-poll-mode-bound-overstates-transience
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d4-windows-unset-path-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d6-socket-limit-is-linux-not-posix
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d7-attach-preflight-shell-resolution
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d8-unmeasured-attribution-presented-as-fact
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d6-d7-rejected-alternative-narrative
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: closes-495-and-lint-comments-not-plan-requirements
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 raised 11 findings against the decision record, all open.
The gating three are D2's inherited `LYX_LOG_FILE` for a detached daemon, D4's zero-pin rebuild also running in the attach pre-flight, and D7's unbounded `-f /dev/null` drop with a nonexistent `reed.yaml` remedy.
The rest are bound and wording corrections, and one relocation (D6/D7 narrative, with D7's never-`-g` constraint lifted first).
