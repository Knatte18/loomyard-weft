# Fixer-report, round 1

## Changes

- **F4** (commit `067ad062`): `internal/fabricengine/parenttips.go` gains `HeadHoldsCommitsNotIn(ref)`, built on `ResolveSHA` and `CommitsNotIn` with no `CommitDetail` read. The `TaskHasCommits` closure in `internal/loomcli/landingdeps.go` calls it instead of `CommitLog`. Fixed as proposed.
- **F1** (commit `252d9165`): `TestLandingDeps_TaskHasCommitsReadsTheTaskHeadAgainstTheParent` in `internal/loomcli/landingdeps_integration_test.go` builds a real task pair and asserts false when level, true after one task commit, and an error naming the parent branch when it does not resolve. Fixed as proposed.
- **F2** and **F3** (one commit, since both edit the same two test files): `prreview_test.go` and `finalize_pr_test.go` gain a `taskErr` field, a `wantErrText` check, a failing-`TaskHasCommits` row in each table, and a rejected-token-in-`url.Error` row in Finalize's table. Fixed as proposed.
- **Self-check follow-up**: `TestHeadHoldsCommitsNotIn` in `internal/fabricengine/parenttips_integration_test.go`, and the loomcli integration test's file header widened to name the PullRequest seams.
- **Gate finding**: the widened header in `internal/loomcli/landingdeps_integration_test.go` was a fixed-column-wrapped sentence, so no test ran.
  It is now one sentence per line, and the tier 1 gate for `./internal/loomcli` passes.

Gates: `lym gate test` tier 1 for `./internal/loomcli` and `./internal/fabricengine`, and tier 0 for `./internal/landingshed`, all passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the loomcli integration test's file header omitted the PullRequest seam tests. Fixed as proposed.
- fix-consistency: LOW, `HeadHoldsCommitsNotIn` had no test in the fabricengine integration file. Fixed as proposed, by adding `TestHeadHoldsCommitsNotIn`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
