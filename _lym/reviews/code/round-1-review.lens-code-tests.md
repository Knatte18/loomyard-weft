```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke.

**Behaviour → pinning test**
1. `checkTaskSetting` takes `false`, `true` and `unattended` (plain or quoted) and refuses `maybe`, `"true"` and `1`, naming the three values and the way forward → `TestAutonomyLevels_EveryCheckingPathTakesTheThreeLevels` (internal/dobbyengine/config_test.go:105). This covers all three paths: `LookupAutonomy`, `LoadConfig` and `CheckLocalValues`.
2. `LookupAutonomy` returns the level, and a refused value reads `AutonomyOff` → the same test (`want` with `wantRefused`).
3. `AutonomyLevel.On` is true for `true` and `unattended` → the same test (`wantOn`).
4. A boolean in a board `run_settings` still reads → the same test, through `CheckLocalValues`, which `configreg`'s CheckValues runs.
5. `lym config beam --set autonomy=unattended` is accepted → covered through `LoadConfig`, the check `CheckLoad` runs. No end-to-end configcli row, which I judge enough.
6. `hubgeom.AutonomyWithRefusal` returns the level, hands back the refusal only for an ErrInvalid failure, and warns once → `TestAutonomy` rows: the `unattended` row, "a refused value reads false, hands back its refusal and warns once", and "a location that is no git worktree … no refusal" (internal/hubgeom/autonomy_integration_test.go:115,133,155).
7. `hubgeom.Autonomy` delegates → indirect only. The batten and loom stamp tests reach it through `WithRunAutonomy`; acceptable for a delegate.
8. `StampAutonomy` carries the level, `unattended` included → `TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy` row `PrimeDobbyUnattended` (internal/beamcli/dobbynoticestamp_integration_test.go:123).
9. A refused value stamps `false` and the orch is told once before the first stamped mail → the same test, row `BeamDobbyRefused` (line 124).
10. The episode ends at the next read that is not refused, and a later refusal posts again → **none**.
11. A failed refusal post leaves the episode untold, so the next refused read posts again → **none**.
12. The board's beam modes show `unattended` → `TestBeamModes` (internal/hubgeom/beammodes_integration_test.go:225,234). `off` and `on` stay pinned by its root rows.
13. `orchAutonomy` maps the three levels → `TestOrchAutonomy_KeepsEachLevel` (internal/orchcli/start_test.go:1700). No public-surface test covers `unattended` through `orchPaths`, so this is not redundant.
14. `WithLaunchFacts` picks the `unattended` line → `TestWithLaunchFacts` rows `UnattendedUnderSlack` and `UnattendedUnderTerminal` (internal/orchengine/prompt_test.go:61,64). This pins a fragment Go chooses, which the rules allow.
15. loomcli `runAutonomy` (loom `ApplyAutonomy`, drift forced keys, `autonomySettingLookup`) reads `unattended` as on → **none**. `TestRunAutonomy_ForcesTheRunsKeysOverARealHub` has only `true` and `false` rows (internal/loomcli/autonomy_integration_test.go:76-79).
16. pawl's `Autonomous` seam reads `unattended` as on → **none** (internal/loomcli/pawlwiring.go:154).
17. The Slack hook's `orchAutonomous` reads `unattended` as on → **none**. hook_test fakes the bool (internal/slackcli/hook_test.go:54,139), so the `.On()` wiring at internal/slackcli/wiring.go:131 is unpinned.
18. Assertions removed or widened → none loosened. The removed `TestLoadConfig` row "a non-boolean autonomy is refused…" (config_test.go:92 at base) is replaced by stricter refusal assertions in item 1's test.

**Findings**

- **MEDIUM**, class `design`, at internal/beamcli/dobbydeps.go:160-180 (`runAutonomyStamps.tellRefusal`).
  - Summary: no test pins how a refusal episode ends or retries. Nothing checks that a non-refused read ends the episode and a later refusal posts again, or that a failed post leaves the episode untold so the next refused read retries.
  - Evidence: internal/beamcli/dobbynoticestamp_integration_test.go:121-124. `BeamDobbyRefused` is the last row, so no read follows it. `post` never fails in that test.
  - Fix: a tier0 test of `tellRefusal` with a stub `post` covering refuse → refuse → nil → refuse (two posts) and failing post → refuse (posted on retry). Or add rows after the refused row in the existing test.
- **MEDIUM**, class `design`, at internal/loomcli/driftwiring.go:102, internal/loomcli/pawlwiring.go:154 and internal/slackcli/wiring.go:131.
  - Summary: no test drives `unattended` through the readers that ask only whether autonomy is on. Reverting any of these `.On()` calls to `== AutonomyOn` would pass every test.
  - Evidence: internal/loomcli/autonomy_integration_test.go:76-79 has only `true` and `false` rows. internal/slackcli/hook_test.go:54 fakes `orchAutonomous` as a bool.
  - Fix: add an `autonomy: unattended` row with `forced: true` to `TestRunAutonomy_ForcesTheRunsKeysOverARealHub`, which covers the loom and drift readers. The pawl and Slack wiring stay one-line seams with no row unless a wiring test reaches them.

Tool calls: none.
