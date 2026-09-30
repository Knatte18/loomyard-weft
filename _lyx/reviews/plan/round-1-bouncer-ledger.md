---
round: 1
ledger:
  - key: exclude-checkignore-raw-gitexec
    rounds: [1]
    status: resolved
  - key: batten-integration-no-driven-path-assertion
    rounds: [1]
    status: resolved
  - key: pathtracked-bundled-in-exclude-card
    rounds: [1]
    status: resolved
  - key: exclude-lock-criterion-untested
    rounds: [1]
    status: resolved
  - key: spawn-helper-missing-prime-returns
    rounds: [1]
    status: resolved
  - key: taskchaininteractive-missing-from-create
    rounds: [1]
    status: resolved
  - key: exclude-written-before-tracked-check
    rounds: [1]
    status: resolved
  - key: verify-skips-cmd-lyx-guards
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings against the first plan draft.
The BLOCKING one was a raw `gitexec.RunGit` that would have broken the Checked-Call pin.
The others were an integration test gap, a granularity split for `PathTracked`, an untested exclude-lock criterion, the shared helper's return values, a missing `TaskChainInteractive` target, the exclude-before-tracked-check order, and verify skipping `cmd/lyx`.
The fixer applied all eight in the same round, which renumbered the cards to 1-6.
The judge checked each fix against the cards and the named code, and approved.
No finding is open.
