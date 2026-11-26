```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails when it breaks.

**Per behaviour: the test that pins it, or none**

`internal/dobbyengine`
- `LookupWithSource` returns the anchor index of the task's own file (0), the parent (1) or the prime (2): pinned by `TestLookupTaskSettings` rows, `wantSource` (tasksetting_test.go:601-610).
- `LookupWithSource` returns `HubWideSource` when the board default layer decides: pinned by the "a hub default that is on/off" rows (tasksetting_test.go:608-609).
- `LookupWithSource` returns `HubWideSource` when the template decides: pinned by the "nothing set anywhere takes the template's values" subtest (tasksetting_test.go:146-148).
- `lookupTaskSetting` delegates to `LookupWithSource`, with values unchanged: pinned by the existing `TestLookupTaskSettings` value assertions.

`internal/hubgeom`
- `TaskSettingChain` names each anchor (slug, or `""` for the prime): pinned indirectly by `TestBeamModes` through `SetBy` `bm-beam` and `root`.
- `Names` stays aligned with `Anchors` when a worktree in the walk is absent on this machine: **none**.
- `BeamModes` reads the hub-wide values, the root's own value and an inherited value: pinned by `TestBeamModes`.
- Autonomy maps `true` to `on` and `false` to `off`: pinned by `TestBeamModes` (root `on`, hub-wide `off`).
- `primeOfBoard` resolves the prime from the board directory: pinned by `TestBeamModes`.
- `primeOfBoard` refuses a records main worktree that has no owner beside it: **none**.
- `readBeamMode` reads the parent branch from the origin record and the PR from the landing record: pinned by `TestBeamModes`.
- `readBeamMode` with the beam's worktree absent (no PR link, chain skips the beam) or no origin record (empty `ParentBranch`): **none**.
- `readBeamMode` with a landing record that holds no pull request: **none**.
- `BeamModes` leaves out a beam whose read fails and keeps the others: **none**.
- `readBeamSetting` names an intermediate worktree as `SetBy` (nested beam, or a non-beam parent): **none**.

`internal/boardengine`
- `Config.Outputs` carries the installed reader: pinned by `TestRenderToDisk_ShowsTheInstalledBeamModes`, which goes through `Config.Outputs()`.
- `readBeamModes` passes the open beams' slugs to the reader: pinned by the `gotBeams` assertion (beammodes_test.go:327).
- `readBeamModes` leaves out done beams and non-beam projects: **none** (the fixture holds only two open beams).
- A reader error renders the board with no mode line and no header: **none**.
- With no reader installed, the board has no mode lines: covered by every existing `RenderToDisk` test, which renders with no reader.
- `modeHeader` writes the hub-wide line: pinned by `wantLines[0]` in both rows. The legend lines are static prose, so they are exempt.
- `modeLines` for the root: pinned by the "Autonomy **on** …" lines.
- `modeLines` for a beam the reader left out of `Beams`, and for a project that is not a beam: **none**.
- `everyPlaceHubWide` drops a setting every place takes from the hub-wide default, and keeps one some place sets: pinned by row 2's `wantAbsent` and row 1.
- `settingText` writes an own value bold and an inherited value italic: pinned by both rows.
- `sourceText` writes `hub-wide` and `P0`: pinned.
- `sourceText` writes a beam's handle (a value inherited from another beam), or a code-span slug for a worktree with no handle: **none**.
- Parent branch and Beam PR are present or left out: pinned (beam-a has both, beam-b has no PR, the root has neither).
- `fileHeader` puts the mode line into `project-&lt;slug&gt;.md` and `project-root.md`: **none**. The test reads `README.md` only, and the mutation survivor at render.go:423 confirms it.
- `Render` writes no mode line even with a reader installed: **none**.

`cmd/lym`
- The pre-run installs `hubgeom.BeamModes`: **none**. The pre-run is skipped under `testing.Testing()`, so no test can reach it.

**Findings**

1. MEDIUM, design, `internal/boardengine/beammodes_test.go:243`. No row covers a value a beam inherits from another beam or from a non-beam worktree. `sourceText`'s handle lookup and code-span fallback are unpinned, and so is `readBeamSetting` naming an intermediate `SetBy` (no nested beam in `TestBeamModes`). Evidence: beammodes.go:215-218 and hubgeom/beammodes.go:796-800 against the fixtures at beammodes_test.go:300-317 and beammodes_integration_test.go:854-864.
2. LOW, design, `internal/boardengine/render.go:456` (`fileHeader`, mutation survivor at render.go:423). No test checks the mode line in `project-&lt;slug&gt;.md` or `project-root.md`; the only new test reads `README.md` alone. Evidence: beammodes_test.go:330.
3. LOW, design, `internal/boardengine/beammodes.go:109`. No test checks that `readBeamModes` leaves done beams and non-beam projects out of the reader's list, or that `modeLines` returns nil for them; the fixture tasks are two open beams only. Evidence: beammodes_test.go:250-253.
4. LOW, design, `internal/boardengine/beammodes.go:113-117`. No test checks the stated contract that a reader error renders the board with no header and no mode line. Evidence: the only reader in beammodes_test.go:296 never fails.
5. LOW, design, `internal/hubgeom/beammodes.go:699-706` and 741-764. No test covers `readBeamMode`'s other paths:
   - a beam whose read fails is left out while the others stay;
   - an absent beam worktree gives no PR link and the beam drops out of its chain;
   - no origin record gives an empty `ParentBranch`;
   - a landing record holds no PR.

   Evidence: `TestBeamModes` has one beam on the happy path (beammodes_integration_test.go:850-899).
6. LOW, design, `internal/hubgeom/beammodes.go:724-726`. `primeOfBoard`'s refusal of a records worktree that has no owner beside it, and the `Names` alignment in `TaskSettingChain` (tasksetting.go:38) when a worktree in the walk is absent, have no test. `TaskSettingChain`'s own tests never assert `Names`. Evidence: grep of `internal/hubgeom/tasksetting*_test.go`.

Tool calls:
1. Grepped `internal/hubgeom/tasksetting*_test.go` for `Names` and read tasksetting_test.go:120-150. My context lacked whether the existing chain tests assert `Names`, and which subtest holds the template-source assertion.
