---
round: 5
focus:
  - "In `_lyx/discussion/decision-record.md`, the Decision that merges tests into a scenario test (Decision 13) with Decision 4's `t.Parallel`-per-subtest rule: settle whether the two can be followed together, given that `t.Run` returns true for a subtest that called `t.Parallel` before that step runs, and what the Decision says about parallelism of the scenario and of its steps."
  - "In the same Decision 13, the merge-eligibility rule for steps sharing one fixture: settle whether it bounds what a step may leave behind for a later step to observe, or whether a comment naming a mutation is enough to merge, since each former test ran on a fresh fixture."
  - "In the same Decision 13, the per-family timing that decides a merge: settle whether the command, package, cache handling, margin against run-to-run spread, tie outcome and load record are stated so noise or a cached run cannot decide a merge."
  - "Decision 8's alone-rerun (`-run '^Name$'`) against a test that Decision 13 merged away: settle whether the Decisions say which name is re-run and in what order the verdict and the merge happen, so the rerun cannot pass vacuously."
  - "Decisions 2 and 12 against Decision 13: settle whether a merge is named among the outcomes for a `Removable: no` candidate and among the explanations for a coverage drop."
---

Round 4 left one gating design finding, about parallel steps in a merged scenario.
The other entries cover the rest of the scenario-merge decision, where the round-4 findings clustered.
