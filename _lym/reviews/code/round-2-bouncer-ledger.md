---
round: 2
ledger:
  - key: memory-max-literal-and-constant
    rounds: [1]
    status: resolved
  - key: gatecli-cap-test-unbounded-hog
    rounds: [1]
    status: resolved
  - key: summary-other-gate-work-unproven
    rounds: [1]
    status: resolved
  - key: holder-end-not-pinned
    rounds: [1]
    status: resolved
  - key: holder-shares-process-group
    rounds: [1]
    status: resolved
  - key: unaligned-cap-untested
    rounds: [1]
    status: resolved
  - key: holder-never-child-untested
    rounds: [1]
    status: resolved
  - key: kept-tests-missing-testtiming-keep
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: holder-test-repeats-cgroup-path-literals
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised seven findings; round 2 confirmed all seven resolved.
Round 2 raised two new BLOCKING `consistency` findings in `internal/proc/memorycap_linux_test.go`.
F8 asks for `//testtiming:keep` markers on two kept tests.
F9 asks for the cgroup path literals in the holder test to use proc's own constants through test aliases.
