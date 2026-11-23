```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that fails if that behaviour breaks.

**Each behaviour and the test that pins it:**

1. **The tmux-script pattern denies:**
   - the incident's subshell command;
   - `tmux attach`;
   - `tmux` after a separator;
   - path-prefixed `tmux`;
   - `exec script`;
   - `tmux` inside `bash -c`.

   Pinned by `TestRows_PatternsDenyAndAllow` (`internal/shellguard/rows_integration_test.go:284-289`). The description-field row at :379 also pins that only the command field matches.
2. **The tmux-script pattern allows:**
   - `grep tmux`;
   - `bash script.sh`;
   - `./script.sh`;
   - `tmuxinator`;
   - a quoted commit message;
   - `lym reed attach`.

   Pinned by the same test, :290-295.
3. **The tmux-script steer opens with the whole-command refusal, and its notice does not.** `TestRows_SteersOpenWithWholeCommandRefusal` (`internal/shellguard/rows_test.go:186-198`) loops over every row from `Rows`, so it reaches the new row.
4. **Claude installs the row exactly when `!interactive`.**
   - `TestBuildSettings_NoAgentHookAndForkGuard` checks that `wantTmuxScript` is 1 for autonomous inputs and 0 for interactive ones (`settings_test.go:651-669`).
   - `TestBuildSettings_DenyToggleMatrix` checks 8 bare entries for autonomous inputs and 7 for interactive ones.
   - `settings-golden.json` holds the steer 16 times: each of the 8 `interactive=false` keys carries it in both its settings and its notice, and no `interactive=true` key carries it.
5. **Claude announces the row exactly when `!interactive`.** The golden notices pin it: they hold the tmux notice only for the 8 `interactive=false` keys, so a mutant that always announces fails on the interactive keys.
6. **Claude's manifest lists the row for an autonomous spec and omits it for an interactive one.** Pinned by `TestPrepare_WritesTheManifest`: the `AutonomousSpecWithSkills` want, and the interactive case's `wantAbsent` (`claudeengine/manifest_test.go:135,167`).
7. **The Claude launch writes one more Bash deny.** Pinned by the counts 9/9/8 in `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`.
8. **Codex `switchInstalled` follows `autonomous`.** Pinned by the `SwitchesOff` row of `TestInstalledGuards_CoverEveryRow`, which drops tmux-script, and the `EverySwitchOn` row, which keeps it (`guards_test.go:43-44`).
9. **Codex `switchAnnounced` follows `autonomous`.** Pinned by `TestGuardEntries_RenderEveryRowWithItsCodexText` (`guards_test.go:98`). There `noticesWithoutPython` is built from the installed rows, tmux-script included, so a mutant that never announces drops the notice and fails. A mutant that always announces is equivalent: the row is installed only when `autonomous` is true.
10. **Codex `Prepare` sets `autonomous: !spec.Interactive`** (`codexengine.go:183`). Pinned by `TestPrepare_WritesTheManifest`, autonomous want against interactive `wantAbsent` (`codexengine/manifest_test.go:102,135`). Also by `TestPrepare_InstallsGuardsFromItsInputs`, whose `wantInputs` now carry `autonomous: true`, and by `TestPrepare_SwitchedToolHooks`, which builds `guardIn` from `tt.interactive`.
11. **Stencil edits** in `darn-fragment-make-change.md`, `darn-fragment-make-change-gated.md` and `gatefix-frame-cold.md`: static prose, exempt.
12. **Doc comment edits** (`shellguard/doc.go`, `claudeengine/doc.go`) and the `pattern/` edit: prose, exempt.

**Findings:** none. Every behaviour the diff adds or changes has a test row that fails when it breaks.

Tool calls:
- grep of the `settings-golden.json` keys, the count of the tmux steer in it, the `TestRows_SteersOpenWithWholeCommandRefusal` body, and how `noticesWithoutPython` is built. My context lacked the golden's interactive keys, whether the steer-opening test loops every row, and how the Codex notice expectation is built.
