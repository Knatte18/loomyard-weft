---
round: 3
focus:
  - "In _lym/discussion/decision-record.md, follow a queued run whose work the parent already holds (D8, and its `landed` outcome bullet) through internal/landingshed/finalize.go (`landUnderLock` with held true, `mergeAndGate`) and settle whether a member the leader landed runs any merge of the parent into its task worktree or any landing gate under the lock, which owner and hand-over checks it still judges, and where its own landing commit comes from when its queue entry is missing."
  - "In the same record's D6 and D8, trace the leader's and a woken member's remaining landing records through `pushParent` and `recordLanding` in internal/landingshed/finalize.go, `RecordLanding` in internal/runpusher/parent.go and `AheadByOnly` in internal/fabricengine/landing.go, and settle whether, after a batch of several runs lands, the push ledger and the dobby's landed-commit record name each run's own landing commit and the next run to take the lock reads the parent as ahead by owed tips alone."
  - "In the same record, settle how a run decides that the parent holds its work when a later member of the same batch rewrote the same lines and the queue entry under .lym is lost, given `HoldsWork` in internal/fabricengine/parenttips.go and the `Lym-Landing` trailer on landing commits: can such a run be landed a second time?"
---

The entries cover what a batched run does once its work is on the parent.
