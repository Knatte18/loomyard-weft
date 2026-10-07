---
round: 4
ledger:
  - key: comment-hard-wrap-multi-sentence
    rounds: [1, 2, 3, 4]
    status: open
    class: scope
    severity: BLOCKING
  - key: watcher-stopping-erased-by-tick-save
    rounds: [1]
    status: resolved
  - key: orch-reload-stencil-comment-stale
    rounds: [1]
    status: resolved
  - key: untagged-tests-missing-parallel
    rounds: [2, 3, 4]
    status: open
    class: scope
    severity: BLOCKING
  - key: reset-unreachable-fallback
    rounds: [2]
    status: resolved
  - key: test-names-contradict-assertions
    rounds: [2]
    status: resolved
  - key: start-watchdog-test-stale-gate-comment
    rounds: [2]
    status: resolved
  - key: watcher-run-done-shadowed
    rounds: [2]
    status: resolved
---

The comment-line-break key is open for a fourth round.
F1 covers four changed comment lines that end mid-phrase, in `internal/loomcli/cli_test.go`, `internal/loomcli/entryobservation.go`, `internal/loomcli/smoke_test.go` and `internal/loomengine/seed.go`.
F2 covers five comment lines that keep two independent clauses on one line, in `internal/orchengine/doc.go`, `internal/loomcli/integration_driverbootstrap_test.go`, `internal/shuttleengine/claudeengine/claudeengine.go` and `internal/orchcli/doc.go`.

The missing-`t.Parallel` key is open for a third round.
F3 lists ten untagged tests edited by more than one line in `internal/orchengine`, `internal/shuttleengine`, `internal/shuttleengine/claudeengine` and `internal/shedadapters`, none parallel and none with a comment naming global state.

The review classes both keys `scope`, where the previous ledger had `consistency`.
Both defect classes recur in newly edited text each round, so the fix needs a sweep of the whole diff, not of the named lines.
The four round 2 design and naming keys and the two round 1 keys stay resolved.
