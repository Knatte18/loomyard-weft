---
round: 6
focus:
  - "decision-record.md, the scenario-merge Decision (13): when several tests are merged into one scenario whose steps are serial `t.Run` subtests, does a bound say what a later step can observe of an earlier step's leftovers (files, refs, commits, worktree contents, sessions, panes, status and lock files), and how a restoring step's restoration is checked, so that a leak fails the scenario?"
  - "decision-record.md, the scenario-merge Decision (13) against `PATTERN-test-speed`: does a bound state that a family is merged only among tests of one tier, so that a merge never moves an `integration` test into a `tmux` file and out of the gates that ran it?"
  - "decision-record.md, the per-family timing rule of the scenario-merge Decision (13): are the spread, the measured time, the load-difference rule, the order of before runs, merge and after runs, and the tie outcome each defined so that measurement noise cannot decide a merge?"
  - "decision-record.md, Decision 8 (alone rerun of a `failed` entry) against Decision 13: does the rerun command defeat a cached result, and does the text say a run that matches no test is not a verdict?"
  - "decision-record.md, Decisions 2, 4, 10 and 13 and the Goal: do they agree on whether a scenario is a folded table for the per-subtest `t.Parallel` rule, whether a protected test may host a scenario, and whether Decision 10's reason class and the Goal's activity list name the merge?"
---

Each entry names a Decision to read and one question to settle.
