```yaml
lens: row-validity
outcome: reported
```

Row-validity lens review of the 10-card plan for Codex developer instructions.

**Findings**

1. MEDIUM — Card 7 (`TestUserSetup_SkillListing`) and Card 8 (`TestUserSetup_PluginSessionStartContexts`): the planned rows cannot reach the skill and hook branches through the helper the cards name.
   - `internal/shuttleengine/claudeengine#writeInstalledPlugins` (`scope_test.go:36-54`) writes `installPath` as the literal `/plugins/<id>` and only `scope: "user"`.
   - No plugin directory holding `skills/*/SKILL.md` or `hooks/hooks.json` can exist at that path.
   - The "project-scope install for another repository" row needs `projectPath`, which the helper never writes.
   - Card 7 lists the helper under `Uses:`, so it is unchanged, and Card 8 names no plugin-tree helper.
   - Fix: put the helper's change (an install path per plugin under the temp dir, an optional project scope with `projectPath`) or a new fixture helper on Card 7's targets, and have Card 8 use it.

2. MEDIUM — Card 10 (`TestPrepare_InstructionFiles`): "with the anchor in a subdirectory" has no fixture support.
   - `repoFixture.addWorktree` (`fixture_test.go:51-65`) writes the codex.yaml local and worktree layers at the worktree root.
   - An engine anchored at a subdirectory reads `<subdir>/_lym/config/codex*.yaml`, finds nothing and falls back to the template's `home: ~/.lym/codex` and `operator_home: ~/.codex`.
   - The test would then prepare the operator's real Codex home, or fail.
   - Fix: the card states that the rows call `repo.writeCodexLayer(t, subAnchor, "")` and `writeWritableRoots(t, subAnchor, "")`, and lists those helpers under `Uses:`.

3. MEDIUM — Card 8 (`TestRunPluginHook` timeout row): the Intent prescribes no way to make the row pass.
   - `RunPluginHook` runs `sh -c <command>` under a context deadline and waits with `cmd.Wait`, while capturing stdout and stderr.
   - When the context kills `sh`, a forked `sleep` child keeps the pipes open, and `Wait` blocks until `sleep` exits. The row "returns an error within the bound" then fails or hangs.
   - Fix: the Intent sets `cmd.WaitDelay`, or kills the hook's process group, so the bound holds.

4. LOW — Card 4 (`TestRender`, the row with `Root` set): the needed helper change is unlisted.
   - `renderFixture.request` (`render_test.go`, `RenderRequest{Anchor, ScratchDir, BoltDir, Excluded, Only}`) and `mustRender` build every request without `Root`.
   - Fix: list `internal/heddleengine#renderFixture.request` (gains `Root`) under `Edit:`, or say the row builds its request by hand.

5. LOW — Card 4 (`TestRender`, `CodexFileOpensWithRootAgents`): the card says the subtest "becomes a table", but it already is one (`render_test.go:324-349`).
   - Its `AgentsWinsOverClaudeMd` row asserts the behavior this card reverses. The card should name that row as the one that changes to "both", so the old expectation is not kept beside the new row.

6. LOW — Card 4 (`TestSeam`, new subtest `TookInRendersTheCodexFileOnAChangedImport`): the "over an unrelated path re-renders neither" half repeats `TookInRendersTheTargetsBuiltFromAChangedPath`'s existing `UnrelatedPath` row (`seam_test.go`, `{"UnrelatedPath", ".", []string{"src/x.go"}, false, false}`).
   - Drop that half, or make the new case a row of the existing table with a setup field for the CLAUDE.md import and a prior `Made`.

7. MEDIUM — Card 4: two stated branches have no row.
   - "The record is rewritten … empty when the target holds no text, so a dropped import stops matching" has no row. A test that drops the import, re-renders, and then fires a take-in on the old path expecting no re-render would fail if the record kept stale paths.
   - "An empty `Root` writes no record" has no row either.

8. LOW — Card 2 (`TestHeddleCLI`, `show agents-prompt` row): the row repeats the engine-level refusal that `TestRender`'s `ShowAndPassagesWriteNothing` row on this card already reaches.
   - `showCmd` passes the error from `Renderer.Show` straight through (`verbs.go` `showCmd` → `c.renderer.Show`), so the CLI row adds no branch. Cut it.

9. LOW — Card 3 (`TestEnsure_StaleBuildRendersTheHeddleFileTargetsIntoEveryWorktree`): this could be an assertion in the existing `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree` (`ensure_integration_test.go:442`), which already runs `Ensure` over `newStaleHub`.
   - It stays a separate test only because it installs the process-wide seam and so cannot run in parallel. The card should say so, or fold it into the existing test.

10. LOW — Card 6 (`TestPrepare_SubagentTerms`): `internal/agentconfig#Load` (`config.go:137`) is called by the rows but not listed under `Uses:`.
    - The rows also look for the passage inside the quoted, TOML-escaped `-c` setting. The card should say they build the expected setting through `developerInstructions` and `tomlString`, not search for raw text.

11. LOW — Card 7 (`TestUserSetup_InstructionFiles`): the walk above `RepoRoot` goes up to the filesystem root from a `t.TempDir()` path.
    - A `CLAUDE.md` in `/tmp` or `/` on the machine would break the exact-list assertion.
    - Fix: `instructionFiles` stops at a bound the test can set (for example the config directory's parent), or the assertion checks only the fixture files and their order.

12. LOW — Card 10, row "`AGENTS.override.md` at the anchor counts instead of the anchor's `AGENTS.md`": the row fails if wrong only when the two files differ in size and the total passes 32 KiB. The row should state both.

Calls made:
- Read `TestRender`'s subtests in `render_test.go` (`ShowAndPassagesWriteNothing`, `CodexFileOpensWithRootAgents`), which cards 2 and 4 extend and my context did not hold.
- Read `seam_test.go`'s fixture and TookIn table, the `heddlecli` refusal subtest, `TestBuildLaunchCmd` and `TestLoadConfig_Template`, all extended by cards and not in context.
- Read `writeInstalledPlugins`, `newStaleHub` and `geometryOf`, the head of the stale-build test, and the `agentconfig.Load` signature, the helpers cards 3, 6, 7 and 8 rely on.
