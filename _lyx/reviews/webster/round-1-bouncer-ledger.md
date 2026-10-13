---
round: 1
ledger:
  - key: batten-build-check-reresolves-executable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: handover-successor-spawn-reresolves-executable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two design findings of the same shape: code that runs after a deploy resolves the on-disk `lyx` path with `os.Executable`, where the path may read `(deleted)`.
One is in batten's `ChildBuildChanged` closure, the other in the hand-over successor spawn.
Both name a fix that resolves the executable once at wiring time, as `goDriverDepsFor` does.
Both remain open until a fresh review sees the fixed text.
