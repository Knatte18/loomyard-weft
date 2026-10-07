---
round: 3
ledger:
  - key: comment-line-breaks-fixed-column
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: new-tests-omit-t-parallel
    rounds: [1, 2]
    status: resolved
  - key: verify-gate-stuck-reason-no-timeout-case
    rounds: [1]
    status: resolved
  - key: revive-once-marker-ignores-history-episode
    rounds: [1]
    status: resolved
  - key: strandnamed-test-duplicates-public-test
    rounds: [1]
    status: resolved
  - key: overview-loom-bullet-two-sentences-one-line
    rounds: [1]
    status: resolved
  - key: untested-driverrow-and-lock-probe
    rounds: [2]
    status: resolved
  - key: resume-skips-scratch-dir-creation
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: markdown-line-breaks-independent-clauses
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, round 2 three, round 3 three.
The BLOCKING consistency findings of rounds 1 and 2 have narrowed: one `TestShuttleWaiting` comment (F2) is the last Go comment instance of the line-break key, and the `t.Parallel` key is resolved.
The `DriverRow` and run-lock-probe coverage gap from round 2 is resolved.
Round 3 adds one gating finding: `lyx loom resume` reads the status lock and probes the run lock without creating the ephemeral scratch directory (F1, `design`, MEDIUM), unlike `approve.go`, `arm.go` and batten's `runLockHeld`.
Round 3 also adds a LOW markdown line-break finding (F3) over changed lines in `docs/overview.md`, `pattern/PATTERN-completion-signal.md`, `contracts/specs/loom-status-spec.md` and `contracts/stencils/orch/orch-template-role.md`.
