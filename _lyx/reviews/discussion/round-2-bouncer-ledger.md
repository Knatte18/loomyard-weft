---
round: 2
ledger:
  - key: d4-bound-false-for-per-worktree-modules
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: seed-only-upper-layers-pruned
    rounds: [1]
    status: resolved
  - key: d6-strip-silently-inherits-equal-values
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-removal-discards-hand-written-local-override
    rounds: [1]
    status: resolved
  - key: d4-rationale-overlooks-env-expansion
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised four design findings.
Round 2 shows the seed-only-pruning and D9 removal findings no longer raised.
The D4 bound finding recurs: round 1 fixed the shell, tmux and tmux_conf keys, and round 2 finds reed.mouse and reed.debug_log, the same enumeration gap, BLOCKING.
The D6 strip finding recurs in a new form: an open map must be compared and stripped as one unit.
Round 2 adds a LOW finding that D4's rationale ignores env-var expansion as an untracked route.
