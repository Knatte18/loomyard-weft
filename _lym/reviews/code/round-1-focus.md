---
round: 1
focus:
  - "internal/hubreconcile TestMain recorders: do the board sync and notifier recorders keep each test's calls apart by board dir, are they safe under concurrent calls, and does each assertion on calls (including the zero-call ones) still fail when the behaviour it names breaks?"
  - "Parallelism: can any parallel test or subtest reach shared state another one changes (worktree seam, reinterpretation registry, notifier, logger, shared hub fixtures, environment or cwd), and do the rows meant to run alone finish before their parallel siblings start? Compare the file header's account of which tests run alone with the code."
  - "Merged or dropped walks: for each removed test, row, retry walk or second walk, does a remaining test pin the same behaviour no looser, as the amended board entry and the PATTERN entries require? Check each equivalence the change description claims, including the single walk replacing the three stale-build rows."
  - "Assertions and names: did any surviving test lose or widen an assertion without a Loosened-Check trailer, or keep a name or doc comment that states a case it no longer walks?"
  - "Change description against the diff: the 'what changed' and 'coverage merged or dropped' claims, which tests still run serially, the net test-line figures, the timing figures and their before/after support, and the claims in the commit messages."
  - "Change description against the rubric's 'Names of the tooling' bullet."
---

Seed focus for round 1.
The amended board entry limits the cut to `internal/hubreconcile`, with `internal/configengine` and `internal/configsync` left unchanged and their times reported.
The diff is test-only, so the main risk is lost or weakened coverage and races introduced by running tests in parallel.
