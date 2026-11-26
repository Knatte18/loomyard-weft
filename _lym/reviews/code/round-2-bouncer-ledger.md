---
round: 2
ledger:
  - key: cancel-rerun-refuses-with-only-origin-records-branch
    rounds: [1]
    status: resolved
  - key: fabric-docgo-describes-batten-cancel
    rounds: [1]
    status: resolved
  - key: records-archive-origin-condition-dropped
    rounds: [1]
    status: resolved
  - key: pair-gone-naming-narrowed-unrecorded
    rounds: [1]
    status: resolved
  - key: pending-record-never-cleared-when-pair-gone
    rounds: [1]
    status: resolved
  - key: pair-remote-background-four-argument-remove
    rounds: [1]
    status: resolved
  - key: remove-held-lock-refusal-rerun-untested
    rounds: [1]
    status: resolved
  - key: pairteardown-rerun-forwarding-untested
    rounds: [1]
    status: resolved
  - key: summary-blast-radius-and-finish-claim
    rounds: [1]
    status: resolved
---
Round 1 raised the keys above, centred on a removal record naming the batten cancel as its re-run in states where the cancel did not finish the teardown.
Round 2 reports each resolved and raises nothing new.
The narrowing of the board entry is settled by the record's added entry "A removal record names the cancel only when a cancel wrote it".

The round 2 review records these observations for the parent, below its threshold and not as findings:

- `## Merge danger` and the new tag push to origin through `SettleGonePairTeardown`.
- The settle drops a gone pair's pending record whatever its `Rerun`.
- The doc of `TestCancelIntegration_FinishesAnUnfinishedRemoval` holds only for in-process remote steps.
- `pattern/PATTERN-pair-remote.md` omits that the guard skips test files and the owning packages.
- The "the unfinished teardowns" probe refusal has no test.
- The `scheduler.Stop()` gate fix in `dobbyengine` is absent from `_lym/landing/summary.md`.
