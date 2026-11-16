```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the added `GroupingProjectWhoseRunFiledItsFamilyResumesOnlyWithTheFilingMark` subtest in `beamland_integration_test.go` for correctness (lens fix-code-correctness).

no findings

Traced for `g`, a project with no beam flag, called with `refuse("g", "run", runShed, true, true)`:
- **Before the mark:** `Marker` finds no cancel, and `Promoted` reads false because `g` is not a task. `Project` reads project=true and beam=false, and `found` is true. `FilingMarked` reads false, so the call returns "g is a grouping project, …". The test's `strings.Contains(…, "grouping project")` holds.
- **After `WriteFilingMark(h.Location, "g")`:** `FilingMarked` reads true and `refuseProject` returns judged=false. The task checks that remain all pass. `Unreleased`, `Raddle` and `PairParts` are skipped because `found` is true. The Worktree-Create check is skipped because the status is Run-Shed. The call returns nil.

The test fails with the defect in place:
- If the `FilingMarked: …` line is dropped from `hubStartGuard`, the field is nil, the refusal persists, and the post-mark assertion fails.
- If the wiring reads another run ID or location, the mark is never seen and the same assertion fails.
- The tier-1 gate log shows the subtest as PASS. The pre-mark assertion passing also proves that `ParseOwners` records `g` as a project.

The changed behaviour is test-only and no signature changed, so no caller can break. The mark stays on `h.Location` for the later subtests, but none of them reads `g`'s mark, and the hub is a per-test copy. The `shedrun` import is used, and `lym loom fmt` reported no change.

Tool calls by the fork: none.
