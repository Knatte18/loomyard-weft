---
round: 1
ledger:
  - key: tier2-fixture-callers-leave-board-sync-running
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: late-pid-never-killed-and-misreported
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: contained-cleanup-test-lacks-keep-marker
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round.

- `tier2-fixture-callers-leave-board-sync-running` (F1): the wait-and-kill cleanup covers the tier1 users of the fixtures only,
  so the tier2 callers of `newWiredPairFixture` can still leave a detached `lym board sync` running.
  The change summary's claim that their own teardown order covers them goes with it.
- `late-pid-never-killed-and-misreported` (F2): a pid appearing after the kill pass is never killed, yet is reported as surviving the kill.
- `contained-cleanup-test-lacks-keep-marker` (F3): the new cleanup test needs a `//testtiming:keep` marker.
