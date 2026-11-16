```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I name the test row that would fail if that behaviour broke.

## Behaviour → pinning test

1. **No pull request recorded, a merge task not done:** pinned by `TestPROpen/OpenMergeTaskRefusedNamingItsStatus` (internal/beamcli/pr_test.go:129). It checks the way forward and that nothing was pushed, written or changed on GitHub.
2. **No pull request recorded, the beam conflicts with its parent:** pinned by `TestPROpen/ConflictWithTheParentRefusedNamingMergeParent` (pr_test.go:137).
3. **Recorded pull request that GitHub reports closed, beam conflicts:** pinned by `TestPROpen/RecordedClosedConflictingWithTheParentRefusedAndNotReopened` (pr_test.go:142). It asserts no `Reopen` call.
4. **Recorded closed pull request, a merge task not done:** no row. The same `judgeOpenable` call guards it, so row 3 only shows that the call sits on that path.
5. **A merge task not done is checked before the conflict, as the doc says:** none. No row sets both a merge task and `parentConflicts`.
6. **A done merge task does not refuse:** pinned indirectly. `openMergeTask` is unchanged, and the mergeparent_test.go:61 row with `Status: &amp;done` covers it.
7. **A recorded pull request GitHub reports open still answers `already` while the beam conflicts:** none. `RecordedOpenAnswersAlreadyAndCommitsWithoutAnotherGitHubCall` (pr_test.go:85) runs with `parentConflicts` false.
8. **A beam only behind its parent still opens:** none. No row sets `parentUnmerged=true` and expects the open to go ahead. Every row's default world has `parentUnmerged` false.
9. **A failed `Tasks()` read or a failed `ParentConflicts` read inside `judgeOpenable` returns a rerun refusal:** none. The fake at land_test.go:167 and land_test.go:207 never returns an error.
10. **`ParentConflicts` over real git, clean case** (a parent commit on another file reads false): pinned by `TestHubLandDeps_ReadyReads` (deps_integration_test.go:76).
11. **`ParentConflicts` over real git, conflict case** (both sides wrote one file apart reads true): pinned by deps_integration_test.go:102.
12. **`ParentConflicts` against the upstream tip read after a fetch:** none. When the conflict check at :102 runs, the upstream is unset (:82), so only the local tip is tested.
13. **`ConflictsWith` keeps a failed merge-tree run or a ref that does not resolve apart from a conflict:** none. fabricengine has no `ConflictsWith` test; `TestHoldsWork` covers only `HoldsWork`.
14. **The `ParentUnmerged` refactor through `beamParentTips`:** pinned by the existing rows at deps_integration_test.go:69, :73, :83 and :87.

## Findings

- **LOW · design** · internal/beamcli/pr.go:242-252. The error paths of `judgeOpenable` (a failed `Tasks()` read and a failed `ParentConflicts` read, each answering a rerun refusal) have no test row. The `landWorld` fakes cannot return an error (land_test.go:167, land_test.go:207).
- **LOW · design** · internal/fabricengine/parenttips.go:222-236. In `ConflictsWith`, a failed merge-tree run or a ref that does not resolve answers an error, which is kept apart from `clean=false`. No test pins that. fabricengine has no `ConflictsWith` test, and the beamcli integration test drives only the clean and conflicted results (deps_integration_test.go:76, :102).
- **LOW · design** · internal/beamcli/pr.go:236-257. Three behaviours have no row that would fail if they broke:
  - The doc comment says the merge-task refusal comes before the conflict refusal (pr.go:237-239), but no row sets both.
  - "A beam only behind its parent passes" (pr.go:240) has no row with `parentUnmerged=true` that opens.
  - A recorded pull request GitHub reports open answers `already` even while the beam conflicts. The `RecordedOpen…` row runs with `parentConflicts` false (pr_test.go:85-100).
- **NIT · design** · internal/beamcli/deps.go:163-167. `ParentConflicts` reading the upstream tip after a fetch is not covered. The conflict check at deps_integration_test.go:100-104 runs after the upstream was unset at :82.

Tool calls: none; the inherited context held every file this lens needed.
