---
round: 1
ledger:
  - key: fallback-envelope-lacks-envelope-path
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: drive-dir-guard-prose-no-bound
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d2-rationale-understates-key-reads
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: record-write-failure-no-acceptance-bullet
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: claude-env-file-narrated-thrice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first ledger; there was no prior ledger to carry forward.

- `fallback-envelope-lacks-envelope-path` (F1): D3 routes `friction_dir`, `scratch_dir` and `next_interrupt_policy` through `envelope_path`, but the D2 fallback envelope has no such key.
- `drive-dir-guard-prose-no-bound` (F2): D3 replaces the structural subshell guard with a prose no-`cd` rule and states no bound for a bare call from a drifted cwd.
- `d2-rationale-understates-key-reads` (F3): the Rationale says the keys are needed only at parking or repair, but the stop report reads them at every stop.
- `record-write-failure-no-acceptance-bullet` (F4): the acceptance criteria cover the empty `StepsDir` fallback but not a failed record write.
- `claude-env-file-narrated-thrice` (F5): `CLAUDE_ENV_FILE` internals are repeated in three places, with a rejected-alternative clause in a Decision.

No finding is gating, so the loop converges at round 1.
