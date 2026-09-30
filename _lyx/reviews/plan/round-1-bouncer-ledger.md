---
round: 1
ledger:
  - key: baseline-startsha-overwritten-on-rebegin
    rounds: [1]
    status: resolved
  - key: stage-wiring-stale-outcome-and-callsite-comments
    rounds: [1]
    status: resolved
  - key: dirty-tree-restore-case-unspecified-dirt
    rounds: [1]
    status: resolved
  - key: no-bisector-notice-listed-unconditionally
    rounds: [1]
    status: resolved
  - key: stencil-card-intent-missing-d11
    rounds: [1]
    status: resolved
---

Round 1 raised five findings against the plan.
The only BLOCKING finding was the D7 baseline:
`BeginBatch` overwrote batch 1's `StartSHA` with the current head on a re-begin, which could excuse a first-fork regression as pre-existing.
The fixer added card 12, which inherits a prior non-empty `StartSHA` the way `recover-batch` already does.
The two LOW findings are fixed in card 9:
it now names the stale `RunResult` and call-site comments,
and it pins the dirt shape for the restore case.
The two NITs are also fixed:
card 9 limits the no-bisector notice to a regression,
and card 10's Intent now cites D11.
The judge found every fix present in the current plan and raised no new finding, so every entry is resolved.
