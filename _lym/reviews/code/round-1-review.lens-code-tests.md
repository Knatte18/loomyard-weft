```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none (lens code-tests).

**Behaviour → pinning test**

- **`claudeimports.Expand`.** Pinned by `TestExpand` in `internal/claudeimports/claudeimports_test.go`.
  - Pinned: relative, nested, code span, fenced block, mid-word `@`, missing file, directory target, five-hop limit, cycle, duplicate listing, `~/` import, unreadable entry.
  - Absolute-path import: only indirectly, through heddle's `OutsideTheWorktree` row.
  - Tilde fences, a longer closing fence, and an unclosed backtick run: none.
- **D5 `developer_instructions` order.**
  - Creel head, files, pointers, listing and contexts: `TestPrepare_UserSetup` and `TestPrepare_InstructionFiles` (pointers sit between files and listing).
  - Creel head then standing passages: `TestPrepare_AppendsStandingPassages`.
  - Subagent passage position relative to the standing passages: none. No test runs an effort and passages together; `TestPrepare_SubagentTerms` builds only `[head, passage]`.
- **D5 empty set omitted.** `developerInstructions(nil)` has no direct test; `TestBuildLaunchCmd` pins the empty-field path only if one of its unseen rows leaves the field empty. Prepare always has a creel head, so the empty case is unreachable through it.
- **D5 128 KiB refusal.** `TestPrepare_AppendsStandingPassages` rows `AtTheArgumentCap` and `OverTheArgumentCap` pin the boundary, size, largest item, way forward, and no prompt file. `TestPrepare_UserSetup` row `GlobalFileOverTheCapRefuses` pins the largest item when it is a source item. Picking the largest among several items: none, since only one item is ever dominant.
- **D6.** Effort, passage, unlisted-effort warning and thread cap are pinned by `TestPrepare_SubagentTerms`. The config bounds are pinned by `TestLoadConfig_RefusesInvalidValues` (`max_subagent_threads`, `plugin_hook_timeout`). The defaults 16 and 10s are pinned by `TestLoadConfig_Template` and `TestPrepare_UserSetup`.
- **D7 `agents-prompt`.** `TestRender` pins `Show` refusal and the `Passages` rows for claude, claude orch and codex. `lym heddle show` goes through the same `Show`, and the heddlecli help text is prose.
- **D7 hub reconcile render.** `TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree` pins the stale walk, the pair call and a failing seam. `TestWorktreeSeam_Events` pins `NotifyWorktreeMadeAt` with no seam, with a worktree, and with a path in none.
- **D7 shipped entries.** The diff does not change them; the plan says `ShippedEntriesArePinned` pins them.
- **D8 hooks.** `TestUserSetup_PluginSessionStartContexts` pins the following.
  - Stdin, env (`CLAUDE_PLUGIN_ROOT`, `CLAUDE_PROJECT_DIR` as `WorkDir`), the hook's own timeout, and the request-timeout fallback.
  - Plain text vs JSON context, and the matcher rows.
  - A failing hook, a malformed hooks file, a nil runner, and a plugin the scope turns off.
  - Warning for an omitted hook: `TestPrepare_UserSetup`.
  - Timeout, kill and wait of the real child: `TestRunPluginHook`.
- **D8 instruction files.** `TestUserSetup_InstructionFiles` pins the global file, above-root files, import expansion, absent file, unreadable global file as an error, and unreadable above-root file as a warning.
- **D8 skill listing.** `TestUserSetup_SkillListing` pins scope-limited plugins, a user skill turned off, a project skill, a disabled scope, a foreign project install, no description, and a malformed plugins file.
- **D8 unresolvable config dir.** The behaviour is no global file, no plugin skills or contexts, project skills and above-root files kept, and one warning naming `CLAUDE_CONFIG_DIR` or HOME. Test: none. Every fixture sets `claudeDir`.
- **D9 cap.** `TestPrepare_InstructionFiles` pins the sum, the default floor, and override-over-AGENTS. A file that cannot be examined refusing the launch: none.
- **D9 pointers.** `TestPrepare_InstructionFiles` pins below the anchor, beside AGENTS, the heddle-marked override, the unmarked override, a CLAUDE.md with an import, a lone CLAUDE.md, and an unreadable directory skipped with a warning.
- **D9 both files in `AGENTS.override.md`.** `TestRender` rows `BothFiles` and `ClaudeMdImport`.
- **D9 import record and re-render.** `TestRender` `CodexFileRecordsItsImports` pins inside, outside, dropped and unwritable. `TestSeam` `TookInRendersTheCodexFileOnAChangedImport` pins no anchor, an anchored worktree, and a malformed record.
- **Seam wiring.** `TestNew_CodexLaunchCarriesTheClaudeUserSetup` pins the source. The `RunHook` wiring: none. `claudeengine.RunPluginHook` could be dropped from the registry entry with every test still green.
- **`Seams.SandboxProbe` injection.** Used by `newEngine` in `fixture_test.go`.

**Findings**

1. MEDIUM, consistency, `internal/shuttleengine/claudeengine/usersetup.go:538-545`: the config-directory-failure behaviour has no test row. This is the parent's Added-after-Discussion entry for card 7. A test needs `claudeDir` unset with `CLAUDE_CONFIG_DIR` and HOME unresolvable. Evidence: every row of `newUserSetupFixture` sets `source.claudeDir` (`usersetup_test.go:513-514`).
2. LOW, scope, `internal/heddlecli/verbs.go:556` (`Root: event.Root` in `renderInto`): no test shows `lym heddle render` passes the worktree root. If it were empty, `recordImports` would drop every path and imports would silently stop re-rendering. The seam and render tests set `Root` themselves.
3. LOW, scope, `internal/shuttleengines/shuttleengines.go:82-85`: the `RunHook: claudeengine.RunPluginHook` wiring is untested. `codexlaunch_test.go` covers only the source. Add a plugin with a hook to that test and assert its output in the launch line.
4. LOW, scope, `internal/shuttleengine/codexengine/instructionfiles.go:543-545`: the stat error that refuses the launch ("examine the instruction file") has no row, only the unreadable-directory skip for the walk. A path failing for a reason other than not-exist is the input.
5. LOW, scope, `internal/shuttleengine/claudeengine/usersetup.go:664-678,708,718` and `pluginhook.go:277-287`: error paths with no row.
   - A plugin with no `installPath`.
   - An unreadable plugins file.
   - An unreadable skills directory or `SKILL.md`.
   - An invalid matcher regex.
   - A non-`command` hook type.
   - The orch role in `MatchesRole`.
   - Project skills in directories above `WorkDir` (only the repo-root `.claude` is tested).
6. LOW, consistency, test economy, `internal/shuttleengine/codexengine/prepare_test.go` `TestPrepare_SubagentTerms`: it asserts substrings of the Codex subagent passage ("Your skills are scribe:prose, scribe:testing."). That is static prose pinned by assertion, which the plan's constraint (no test pins a stencil's prose) discourages. Keep the effort and thread-cap assertions and drop the wording ones.
7. LOW, consistency, test economy (report candidates):
   - `TestBuildLaunchCmd` (`command_test.go`) pins the literal flag order and quoting of the new settings. `TestPrepare_Launch` and `TestPrepare_SubagentTerms` compare against `buildLaunchCmd` itself, so they cannot see the literal. Mark it `//testtiming:keep` with that reason.
   - `TestLoadConfig_Template` pins the full resolved `Config` literal, including the new defaults. Mark it `//testtiming:keep` for the template's whole key set.
   - `TestPrepare_ExpandsHomeInWritableRoots`: I could not verify the covering tests' `~/` assertions within the tool budget. If `TestPrepare_Launch` and `TestPrepare_SubagentTerms` do not set a `~/` writable root, mark it `//testtiming:keep` for the `~/` expansion in the launch line.

For the handler, outside this lens: the record's D5 lists the subagent passage first in Claude's file order, while `Prepare` puts the creel head first (`codexengine.go:157-166`), and no test builds the two together.

Tool calls: none; my context already held every diff the lens needed.
