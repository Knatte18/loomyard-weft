---
round: 1
ledger:
  - key: run-exit-cross-check-ignores-dispositions
    rounds: [1]
    status: resolved
  - key: parent-write-split-overlaps-run-state
    rounds: [1]
    status: resolved
  - key: goto-budget-reset-and-jump-entry-undecided
    rounds: [1]
    status: resolved
  - key: invariant-unscoped-violated-on-landing
    rounds: [1]
    status: resolved
  - key: goto-reentry-gate-deferred
    rounds: [1]
    status: resolved
  - key: consumed-correctness-parent-write-unaudited
    rounds: [1]
    status: resolved
  - key: rebaseline-misses-regrouped-batches
    rounds: [1]
    status: resolved
  - key: verify-cannot-run-overlaps-io-row
    rounds: [1]
    status: resolved
  - key: goto-running-reads-as-crash
    rounds: [1]
    status: resolved
  - key: per-row-test-criterion-class-mismatch
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: resolved
  - key: run-exit-undispositioned-findings-no-disposition
    rounds: [1]
    status: open
---

# Cross-round ledger

Round 1 raised eleven findings (F1–F11), and the artifacts as judged already carry a fix for each, so all eleven are resolved.

One gap remains open, and it is the run-exit residue of F1.
D8 now passes dispositioned findings at run exit, but a finding no record-batch ever saw still reaches the cross-check with no stated disposition.
Such findings come from the integration fork's transcript and from Master's activity after the last batch.
Read literally, a policy finding hard-fails a done run, and a correctness finding halts with no way forward.
Round 2 should check that D8 decides both cases and that a must-cover scenario exercises the policy case.
