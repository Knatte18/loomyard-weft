---
round: 1
ledger:
  - key: git-dir-symlink-resolution-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: writable-roots-home-expansion-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW design findings, both missing test coverage in `internal/shuttleengine/codexengine`.
F1 maps to `git-dir-symlink-resolution-untested`, the unpinned `EvalSymlinks` of the git dir and common dir in `readCheckout`.
F2 maps to `writable-roots-home-expansion-untested`, the unpinned `~/` expansion in `configuredWritableRoots`.
Neither gates.
