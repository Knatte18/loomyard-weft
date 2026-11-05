```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens, listing each added or changed behaviour with the test that pins it.

**Behaviour → pinning test**

| Behaviour | Pinned by |
|---|---|
| Refuses a task entry, naming `lym batten run task1` | `TestMergeParent/TaskRefusedNamingItsOwnLanding` |
| Refuses a grouping project, naming `lym beam create group` | `TestMergeParent/GroupingProjectRefusedNamingCreate` |
| Refuses a done beam | `TestMergeParent/DoneBeamRefused` |
| Refuses a beam with no parent branch, naming `lym fabric reconcile` | `TestMergeParent/BeamWithNoParentBranchRefusedNamingReconcile` |
| Refuses a beam with an open merge task, naming `lym batten cancel … --remove` | `TestMergeParent/OpenMergeTaskRefused` (a numbered `-2` slug also exercises the regex) |
| A done merge task is not open, and the next free number is taken | `TestMergeParent/DoneMergeTaskTakesTheNextNumber` |
| Nothing is filed on a refusal | the `len(w.filed) != 0` check in every refusal row |
| Perch turned on from an off budget (1 round, hard cap 2) | `OffPerchTurnedOnForOneRound` |
| Hub budget kept (3 and 5) | `HubBudgetKept` |
| Filed task fields: slug, kind, project, recipe `darn`, `run_settings` | the `fields[...]` check at `mergeparent_test.go:106` |
| Body names `lym fabric merge-in main` and "reviews the merge alone" | `mergeparent_test.go:109` (prose exempt) |
| Result envelope: `task`, `parent_branch`, `way_forward` | `mergeparent_test.go:112` |
| Beam slug cut to fit `MaxSlugLength` | `LongBeamSlugCutToTheBoardsLength` |
| Labels copied from the beam entry | `TestMergeParentCarriesTheBeamsLabels` |
| Default beam buttons `merge-in` and `land` with their confirm prompts | `config_test.go:105-109`, `TestLoadConfig_TemplateDefaultsResolve` |
| Prime group lacks these two buttons and does not fall back to beam | `wantPrimeButtons` in the same test, an exact list, unchanged and still passing |
| Both templates carry the buttons | none seen for windows. The test resolves only the current OS's template. The loop over both files at `config_test.go:126` may be a parity check, but I did not read it, so I can't say if it covers these two buttons. |
| Absent entry, no pair here, and a missing or non-beam origin record refusals | shared `judgeLandEntry`, covered by the existing land tests. I did not open those tests. |

**Findings**

1. **MEDIUM, design.** `internal/beamcli/deps.go`, the new `hubLandDeps` `Tasks`, `FileTask` and `CodeReviewCaps`: no test has them in view. `mergeparent_test.go` and `land_test.go` use fakes, and the integration tests (`dobbydeps_integration_test.go:177-235`) call only `markReady`.
   - `FileTask` does `UpsertTask` and then `SetStatus(slug, nil)`, which releases the task. A task left in draft would never be started by dobby. Nothing fails if the release step is dropped or reordered.
   - `CodeReviewCaps` reads the prime's loom config. Nothing pins that it reads the right config or perch.

2. **LOW, design.** `mergeParentCmd` in `cli.go`: no test reaches the cobra wiring (registration, `ExactArgs(1)`, the `onHubLandDeps` verb string). A grep of the test files found no test using `mergeParentCmd`, and no other test file contains `merge-parent`.

3. **LOW, design.** `mergeparent.go`: the failure paths of `ReadOwners`, `OriginRecord`, `Tasks`, `CodeReviewCaps` and `FileTask` have no row.
   - The `FileTask` refusal carries the way-forward text, and no row shows the wrapped error is returned or that nothing is reported filed.
   - The other four are plain error wraps and matter less.

4. **NIT, design.** `mergeparent_test.go`: no row has a long beam slug whose merge task takes a number above 1 (slug cut and `-2` suffix together), and the `title` field is not asserted.

**Tool calls (1)**
- Grep over the test files for `merge-parent`, `hubLandDeps` and the template names, plus the start of `TestLoadConfig_TemplateDefaultsResolve`. My context lacked the integration tests and the cobra wiring tests, and lacked how the reed test treats the windows template.
