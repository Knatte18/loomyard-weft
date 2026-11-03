```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1 fix, the new `checkBeamWithoutReadableOriginOpensBareFolder` step and its registration in the scenario.

**Walk-through**
- **Missing lock directory row.** The marked beam's origin record exists and `.weft` is gone, so `ReadOriginFor` returns the "lock directory is missing" error at `origin.go:134`. `isBeam` logs a Warn and returns false. `seed` returns the bare anchor, `Spawn` launches it and returns nil. This is the `ReadOriginFor` error branch of `isBeam`.
- **Restore.** `MkdirAll(lockDir)` recreates the directory. The lock file inside it is not restored, but `state.ReadJSON` takes its lock path under the existing directory. The later hub regenerations in the prime steps read this pair's origin record, and the tier1 gate passed with the step running, so the restore works.
- **Absent record row.** After `os.Remove` of the record, `ReadOriginFor` returns `found == false` and no error, so `isBeam` returns false and `Spawn` launches the bare folder. This is the `found == false` branch.
- **Final assertions.** `*launched` equals `[bareFolder, bareFolder]`, and `BeamWorkspacePath` is absent because the beam branch never runs. The step passed in the tier1 gate: its log mentions the step name or slug.

**Regression check.** The test fails if the defect is in place in the error row. If `isBeam` returned true on error, `seed` would regenerate and launch the beam workspace, which breaks both the launch-path assertion and the file-absent assertion. The absent-record row fails only if `isBeam` returns true for a missing record, not if `found &&` is dropped, because a missing record yields a zero `Origin` whose `Beam` is false. That row therefore pins the contract but not the `found` guard itself.

**Callers.** No signature changed; the fix only adds a test function, a step and two imports (`slices`, `lymcwd`).

**Findings**
- LOW: `spawn_hub_integration_test.go`, `filepath.Join(fabricengine.RecordsWorktreePath(pair, slug), ".weft")` hardcodes the literal of the unexported `weftLockDirName` (`recordsgit.go:26`). A rename in `fabricengine` would turn the error row into the absent-lock case without failing: `RemoveAll` of a nonexistent path succeeds, and the later `MkdirAll` would create a stray directory. Nothing in the test pins the literal.

No BLOCKING or MEDIUM findings.

**Tool calls beyond context:** none.
