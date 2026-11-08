---
round: 1
ledger:
  - key: os-bound-subtest-registered-on-windows
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: resize-hook-zoom-skip-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: leave-path-repin-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all about the test coverage of the resize hook in `internal/reedengine`.
F1 (BLOCKING) asks that the OS-bound subtest be left out before `t.Run` on Windows.
F2 (MEDIUM) asks for a test that a zoomed window keeps its zoom through a resize.
F3 (LOW) asks that the re-pin after a strand leaves a shared window be pinned by a test.
All three are open.
