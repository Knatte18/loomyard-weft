```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it (lens: code-tests):**

1. **Beam Spawn launches `BeamWorkspacePath` and not the bare folder.** Pinned by `checkBeamOpensBeamWorkspace` (`spawn_hub_integration_test.go`, the `launched == [beamPath]` assertion).
2. **Beam Spawn regenerates the beam file.** Pinned by the same check. It removes the file after Seed, then Spawn must recreate it, and the check reads it back with the beam as first folder.
3. **Beam Seed regenerates the beam file and launches nothing.** Pinned by the same check: the launcher records no calls and the beam file has the title `TST: <slug> workspace`.
4. **Window command runs once after a beam Spawn.** Pinned by the `windowCommandRuns == 1` assertion at line 236.
5. **`windowDir` is the hub directory for a beam file.** None. `windowCommandRuns` counts runs only, and the marker is not asserted to carry a directory. I saw no assertion on the window directory in any test, so `windowDir` could revert to the anchor path without a failure.
6. **`isBeam` with a record that is found but not marked (task slug) opens the bare folder.** Pinned by the existing `checkTaskSlugOpensBareFolder`, assuming `hubforge.AddPair` writes an unmarked record.
7. **`isBeam` with a missing record (`found == false`) opens the bare folder.** None. No test row shows a slug without a record.
8. **`isBeam` with an unreadable origin record logs a warning and degrades to the bare folder.** None. The grep found no test that makes the record unreadable. The Spawn doc comment claims this behaviour.
9. **A regeneration error on the beam path is returned and never degraded to the bare folder.** None for a beam. The error path is shared with the prime, but I saw no row that fails regeneration for either slug.
10. **Task Seed leaves workspace files untouched.** Not changed by the diff. The beam step runs before the task and prime steps, so any state it leaves is not asserted against.

**Findings**

- **F-T1, LOW, `scope`.** Location: `internal/ideengine/spawn.go:112-120` (`isBeam`), no test. Summary: the unreadable-record and missing-record branches of `isBeam` that fall back to the bare folder have no row. Evidence: the spawn_hub_integration_test.go grep shows no origin-failure case. Fix: add a row to the beam step, or a table in the same file, with a beam slug whose origin record is deleted or whose `.weft` lock directory is removed, asserting the launcher gets the bare folder and no error is returned.
- **F-T2, LOW, `scope`.** Location: `internal/ideengine/spawn.go:102`, `windowDir = filepath.Dir(result.launchPath)` for a beam. Summary: no assertion pins the window directory of a beam launch. Evidence: `spawn_hub_integration_test.go:236` checks only the run count. Fix: make the window-command stub in `setDesktopWindowCommand` record its directory argument, and assert it equals `h.Path` after the beam Spawn. Skip this if the prime check already asserts the same recorded directory, since the row then belongs there.

**Tool calls**

1. grep of `internal/ideengine/*_test.go` for window-command, origin and unreadable coverage. My context held only the diff of the test file, not the rest of it.
