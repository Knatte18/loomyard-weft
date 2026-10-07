---
round: 2
ledger:
  - key: comment-hard-wrap-multi-sentence
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: watcher-stopping-erased-by-tick-save
    rounds: [1]
    status: resolved
  - key: orch-reload-stencil-comment-stale
    rounds: [1]
    status: resolved
  - key: untagged-tests-missing-parallel
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: reset-unreachable-fallback
    rounds: [2]
    status: open
    class: design
    severity: LOW
  - key: test-names-contradict-assertions
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: start-watchdog-test-stale-gate-comment
    rounds: [2]
    status: open
    class: scope
    severity: LOW
  - key: watcher-run-done-shadowed
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised three findings.
Round 2's fresh review found no defect in the `WatcherStopping` save path or the orch stencil header comments, so those two keys are resolved.
The hard-wrapped-comment key reopened: five files still carry mid-phrase or two-sentence comment lines (F1), now `design` and BLOCKING.
Round 2 added five keys: new untagged tests without `t.Parallel` (F2, BLOCKING), an unreachable reset fallback (F3), two contradicting test names (F4), a stale gate comment in `start_watchdog_test.go` (F5), and a shadowed `done` in `Watcher.Run` (F6).
