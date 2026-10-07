---
round: 4
ledger:
  - key: comment-line-breaks-fixed-column
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: BLOCKING
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
    status: resolved
  - key: markdown-line-breaks-independent-clauses
    rounds: [3, 4]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 raised six findings, round 2 three, round 3 three, round 4 two.
The Go comment line-break key has been open every round: each round the reviewer found another clause form (fixed-column wrap, semicolon and `, and`, now `, so`) in a broader set of files, and each fix cleared only the listed lines.
Round 4 files it at BLOCKING (F1).
The markdown line-break key opened in round 3 and is open again at BLOCKING (F2), with sites in `docs/overview.md`, `pattern/PATTERN-completion-signal.md`, `contracts/specs/loom-status-spec.md` and `contracts/stencils/orch/orch-template-role.md`.
The round-3 design finding on `lyx loom resume` and the scratch directory is resolved: round 4 confirmed the directory is created before the status lock and the run lock are used, with a test row for it.
