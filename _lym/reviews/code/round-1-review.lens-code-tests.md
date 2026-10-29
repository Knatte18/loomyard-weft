```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour: pinning test**

1. `WorkspaceSettings` look keys (title, title bar, activity bar), the project colour kept, green as the default, and other keys kept.
   - Pinned by `TestWorkspaceSettings` in `internal/vscode/workspace_test.go`.
   - Rows used: PrimeSettingsKeepTheirKeysAndGreen and BeamSettingsKeepTheBeamColorOnTheTitleBar.
2. `WorkspaceSettings` with missing settings, comments-only settings, and a BOM.
   - Pinned by the NoSettingsTakeGreen, CommentsOnlyTakeGreen and JSONCWithBOMDecodes rows.
3. `WorkspaceSettings` on undecodable JSONC, returned unchanged.
   - Pinned by the `UndecodableReturnedUnchanged_*` rows: truncated, array, null.
   - Nothing asserts the warning is logged. This is minor.
4. `WorkspaceColor` is outside the palette.
   - Pinned by `TestWorkspaceColorIsNoPaletteColor` in `internal/vscode/color_test.go`.
5. The hub and beam files carry the look. The hub title is `<SHORTNAME>: <RepoName> workspace` and the beam title is `<SHORTNAME>: <slug> workspace`. The beam keeps its own title-bar colour and the hub gets green.
   - Pinned by the settings-source subtest of `TestRegenerateHubWorkspace` in `internal/fabricengine/hubworkspace_integration_test.go`.
   - That subtest checks title, title bar and activity bar for both files.
6. `workspaceWindowTitle`'s fallback to `<name> workspace` when the board records no shortname.
   - **None.** Every test in the diff runs with the `TST` shortname.
7. The prime's settings are spliced into the hub file with the look.
   - Pinned by `checkPrimeSplicesSettingsAndRegenerates` in `internal/ideengine/spawn_hub_integration_test.go`. It checks the tab size, title, green and workspace colour.
8. An unchanged regeneration rewrites no bytes.
   - The prior idempotence assertion in `checkPrimeSplicesSettingsAndRegenerates` presumably still covers it. The diff does not show it.
   - `json.MarshalIndent` over a map sorts keys, so the output is stable.
9. `Seed` of the prime writes the hub workspace file, and launches nothing.
   - Pinned by `checkPrimeSeedWritesHubWorkspace`, via the explicit prime slug. It asserts only the title.
10. `Seed` of the prime with no slug (`lym ide seed` run in the prime).
    - **Weak.** The `idecli` "seed without a slug" subtest asserts `settings.json`, the slug and the keybindings outcome, but not that the hub workspace file is written.
    - The no-slug and explicit-slug paths converge on `ideengine.Seed`, so `checkPrimeSeedWritesHubWorkspace` mostly covers it.
11. `Seed` of a task pair leaves the workspace file's bytes untouched.
    - Pinned by `checkTaskSlugOpensBareFolder`.
12. `Seed` returns a regeneration error on the prime.
    - **None for `Seed`.** `checkPrimeWriteFailure` appears to drive `Spawn` only; the diff shows no `Seed` call there.
13. `Seed` seeds the keybindings block into a missing file and returns `created`.
    - Pinned by `checkTaskSlugOpensBareFolder`, which asserts the outcome and that the file exists.
14. `Seed` on an unmergeable `keybindings.json` is reported as skipped and never fails the seed.
    - **None for `Seed`.** The merge logic is shared through `seedKeybindings`, so existing `Spawn` or `vscode` skip tests probably cover the shared seam.
15. `lym ide seed` prints the outcome under `keybindings`.
    - Pinned by the "seed without a slug" subtest in `internal/idecli/cli_test.go`, which asserts `outcome == created`.
16. `slugVerbCommand` change: `open` returns no fields, so its output keeps just `slug`.
    - Covered only by the existing open subtest. The diff does not show it asserting the output.
17. Tests keep `KeybindingsPath` on a temp file.
    - Holds in `cli_test.go`. The dispatch subtest sets a temp path that persists until the parent's deferred restore, and the no-slug subtest sets its own.

**Findings**

- **[LOW] [design]** `workspaceWindowTitle`'s no-shortname fallback has no test.
  - The decision record D2 and its acceptance criteria name this fallback.
  - Add a row, or a subtest in `TestRegenerateHubWorkspace`, with a board that has no shortname, asserting the title is `<name> workspace`.
  - Evidence: `internal/fabricengine/hubworkspace.go` (`workspaceWindowTitle`); no non-`TST` title assertion in `hubworkspace_integration_test.go` or `spawn_hub_integration_test.go`.
- **[LOW] [design]** A `Seed` regeneration failure on the prime is untested.
  - D4 says `Seed` returns the error as `Spawn` does.
  - `checkPrimeWriteFailure` shows no `Seed` call.
  - Evidence: the `seed` error branch in `internal/ideengine/spawn.go`, and `checkPrimeWriteFailure` in `spawn_hub_integration_test.go`.
- **[NIT] [design]** Two `Seed` behaviours rest on shared seams rather than their own rows: the skipped, non-failing keybindings outcome, and the hub file being written when `lym ide seed` runs in the prime with no slug.
  - Evidence: `cli_test.go` lines 105-137; the `Seed` tests in `spawn_hub_integration_test.go`.

**Test economy:** `checkPrimeSeedWritesHubWorkspace` and the title assertions in `checkPrimeSplicesSettingsAndRegenerates` overlap in the title check. Each exercises a distinct entry point (`Seed` versus `Spawn`), so I do not file it.

**Tool calls (1)**
- Ran `grep` on `internal/idecli/cli_test.go` for `KeybindingsPath`, `t.Run` and `Parallel`, to confirm tests keep `KeybindingsPath` on a temp file; the diff context lacked the surrounding lines.
