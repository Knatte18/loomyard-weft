```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F7, F9, F10 and the carried-out parts of F4 and F8), looking for a case where a proposal is wrong or breaks something.

Fork findings:

1. **MEDIUM** — `TestPairTaskBranch_ReadsThePairItself`, rows `OnlyTheRecordsWorktreeStands` and `TaskWorktreeOnAnUnrelatedBranchIsNotRead` (`pairbranch_integration_test.go`). The rows the review's F2 proposed cannot tell the records-worktree step from the local-branch step, so they would still pass with that step deleted. F2 asks the test to pin that step.
   - **Evidence:** in `OnlyTheRecordsWorktreeStands` only the task worktree is removed. Local `pt-pair` and `pt-pair-weft` still exist, so the local-branch loop in `pairTaskBranchLocal` returns `pt-pair` via `exists || weftBranchExists(...)`. That is the same answer the records step gives.
   - The unrelated-branch row is the same. The task worktree is skipped as `elsewhere`, and the records step and the local-branch step both answer `pt-pair`. A bug in the records step would not fail either row.
   - **Fix that discriminates:** add a row where the records worktree stands on `hanf/pt-pair-weft`, for example by creating the branch and checking the records worktree out onto it. The records step then answers `hanf/pt-pair`, while the local-branch loop would answer `pt-pair`, because only the bare branches exist locally.

2. **LOW** — F4's `removePair` change (`remove.go`). Nothing tests the claim the fix exists for. No test pins that a launcher-only pair makes no `ls-remote`, or that an origin-only `Remove` asks origin once. The new tests pass identically with the old double lookup, since the result is the same. This is only noted: the review did not ask for a test on F4, so it is not a defect of the fix.

Every other proposal held up when I tried to break it:
- **F5:** `PairTaskBranchOrOrigin` asks origin only when the local read found nothing, so a pair with any local remnant makes no network call. A prefixed name on origin wins over a bare one through `firstOnOrigin`'s name order.
- **F6:** the skipped records step falls through to the local-branch step, which `weftBranchExists` tolerates. No row can reach the `RecordsRepoRoot` failure, as the fixer will report.
- **F7:** the pull request close test fails under the old `cfg.BranchPrefix + slug` derivation, which would log `hanf/pr-bare`.

Tool calls: none. My context already held every changed file.
