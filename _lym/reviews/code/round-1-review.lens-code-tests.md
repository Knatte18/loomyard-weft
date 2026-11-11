```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it:**

1. **`freeMergeTaskSlug` skips a held run-directory or branch slug, and numbers past -2.**
   - Test: `TestMergeParent/CompletedRunAndLeftoverBranchSkipped` (`internal/beamcli/mergeparent_test.go:65-69`).
   - It holds `b-merge-in` and `b-merge-in-2` and expects `b-merge-in-3`, so ignoring `held` fails it.
   - The test does not separate a held run directory from a held task branch, because `held` is one merged list. `freeMergeTaskSlug` treats both the same, so no separate row is needed.

2. **An error from `HeldSlugs` refuses the verb with a way forward.**
   - Test: `TestMergeParent/UnreadableHeldSlugsRefused` (`mergeparent_test.go:70-73`).
   - It asserts the context text, the wrapped cause and the `lym beam merge-parent b` retry hint.

3. **A truncated long beam slug plus the `-merge-in-N` tail, skipped against `held`.**
   - Existing row: `LongBeamSlugCutToTheBoardsLength`, which pins the cut only against board entries.
   - No row combines a long slug with `held`. The skip logic is the same single map lookup, so this is not a separate behaviour.

4. **`RunIDs` lists directories in name order, skips plain files, and lists none for a missing shed directory.**
   - Test: `TestRunIDs`, rows `AbsentShedDirectory` and `DirectoriesListedFilesSkipped` (`internal/shedrun/paths_test.go:346-384`).
   - The test does not cover a `ReadDir` failure other than "not exist", such as the shed path being a file. The code only returns that error, so it is not a defect.

5. **The `fixloop.go` refactor, with `RunRecordSlugs` now calling `shedrun.RunIDs`.**
   - The fix-loop test (`internal/adoptcli/fixloop_test.go:63`) stubs `RunRecordSlugs`, so nothing drives the real `hubFixDeps` closure.
   - `TestRunIDs` covers the shared reader's behaviour.
   - The old reader's directory was `filepath.Dir(RunDir(prime, "-"))`, the same shed directory `RunIDs` now reads directly, so the refactor preserves behaviour.

6. **`hubLandDeps` `HeldSlugs` wiring: run directories plus `fabricengine.TaskBranchSlugs(prime, "")`, with an error from either source propagated.**
   - Test: none.
   - `deps_integration_test.go` and `dobbydeps_integration_test.go` drive other `hubLandDeps` seams against a real hub, and none calls `HeldSlugs`. `mergeparent_test.go` fakes `HeldSlugs`.
   - Nothing would fail if the wiring dropped the branch half, dropped the run half, used a non-empty prefix, or swallowed an error.

**Findings:**

- **F-tests-1: MEDIUM, class `scope`.**
  - Location: `internal/beamcli/deps.go:242-252`.
  - Summary: the `hubLandDeps` `HeldSlugs` wiring has no test. The fake `HeldSlugs` in `landWorld` hides a wiring regression, such as dropping the task-branch half, a wrong prefix or a swallowed error.
  - Evidence: the grep found no test calling `hubLandDeps(...).HeldSlugs`. The existing `deps_integration_test.go` already builds a real hub pair for other `hubLandDeps` seams (lines 45, 131, 191), so a row there fits.
  - Fix: add a `HeldSlugs` case in `deps_integration_test.go` that makes a run directory and a task branch, then asserts both slugs come back.

Nothing else is a finding. The `TestRunIDs` and `TestMergeParent` rows do not duplicate each other.

**Tool calls:**
1. `grep -rn "hubLandDeps\|HeldSlugs\|RunRecordSlugs\|shedrun.RunIDs" --include=*_test.go internal`: my context did not show whether any test reaches the real `HeldSlugs` wiring or the `fixloop.go` `RunRecordSlugs` closure.
