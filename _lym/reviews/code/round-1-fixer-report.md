# Fixer-report, code review round 1

Every finding is fixed as proposed unless marked otherwise.
Gate: `lym gate test --upto 1` over `codexengine`, `claudeengine`, `shuttleengines` and `heddlecli` passed, and over `codexengine` again after the self-check fixes.
Commits grouped findings that share a file, since `git add -p` is unavailable:
F4, F5 and the instruction-file refusal row of F6 share `instructionfiles.go` and its test;
F3 and the rest of F6 share `claudeengine/usersetup_test.go`;
F7 and F8 share `prepare_test.go`.

## Changes

- **F1** `codexengine.go`: `Prepare` hands the user setup the symlink-resolved `workDir`.
  `TestPrepare_UserSetup` gains the row `AnchorThroughALinkGivesTheResolvedWorkDir`.
- **F2** `docs/overview.md`: the codex engine paragraph now describes the developer-instructions carrier and its cap,
  the subagent terms, the user setup, the project doc cap and the override file, and the pointers.
- **F3** `claudeengine/usersetup_test.go`: new `TestUserSetup_UnresolvableConfigDir`.
  It is its own test, not a row, since `t.Setenv` forbids parallel and the table runs in parallel.
  This differs from the review's wording only in placement.
- **F4** `instructionfiles.go`: `claudeFileIsUnread` applies the heddle exemption for the working directory before the import branch.
  New row `ClaudeFileWithAnImportBesideAHeddleRenderedOverrideIsNotNamed`.
- **F5** `instructionfiles.go`: the unread scan skips the directories of the new constant `unreadScanSkippedDirectories` (`.git`, `node_modules`, `vendor`).
  New row `ClaudeFileInADependencyDirectoryIsNotNamed`.
- **F6**:
  - the registry's hook runner wiring: new tier 1 test `TestNew_CodexLaunchCarriesAPluginSessionStartContext` in `shuttleengines/codexhook_integration_test.go`, over a fixture helper shared with the existing test;
  - `lym heddle render` passing `Root`: new subtest `RenderRecordsTheImportsInsideTheWorktreeRoot` in `heddlecli/cli_integration_test.go`;
  - the examine-instruction-file refusal: row `InstructionFileThatCannotBeExaminedRefuses`, made with a self-linking `AGENTS.override.md`;
  - rows for an unreadable plugins file, a plugin with no install path, an unreadable SKILL.md, an invalid matcher and a non-command hook type in the claudeengine tests.
- **F7** `prepare_test.go`: `TestPrepare_SubagentTerms` drops `wantPassage` and `wantAbsent` and keeps the effort value and the skill names.
- **F8** `command_test.go`, `prepare_test.go`: `testtiming:keep` lines above `TestBuildLaunchCmd`, `TestLoadConfig_Template` and `TestPrepare_ExpandsHomeInWritableRoots`.
- **F9** `heddlecli/verbs.go`: the dropped clauses of the `set` help text are restored.
  They are still true: `renderEveryWorktree` walks every live worktree and warns on a failed listing.

## Gate repair

The gate refused four comment lines that wrap at a fixed column or reach 120 characters, so no test ran.
Each is split at sentence boundaries:
`worktreeseam_integration_test.go` (`requireCloneMadeEvent`), `codexengine/doc.go` (the unread-scan walk passage),
`prepare_test.go` (`TestPrepare_SubagentTerms`) and `shuttleengines/codexlaunch_test.go` (`newCodexLaunchFixture`).
The `worktreeseam_integration_test.go` line was not added by this round's fixes; it is repaired as the gate asked.

A second gate run failed `TestCLIForm_EveryCommand`: the restored F9 clauses took the `lym heddle set` Long text to 2049 bytes, over the 2000 cap.
The repair drops the `house-deny` example line from that help, since the prose above it already describes the claude-settings target.
F9 still holds: the restored clauses stay, and the help text gives up an example instead.

## Self-check

- fix-consistency, LOW: `doc.go` still said the walk skips `.git` alone. Fixed as proposed; the passage names the new skip set.
- fix-consistency, LOW: the `doc.go` heddle-marker passage stays true; no edit. Fixed as proposed (nothing further to change).
- fix-code-correctness, LOW: the skip could hide pointers for a session anchored in a `vendor` directory. Fixed with the same change as the proposals lens below.
- fix-proposal, MEDIUM: the overview said `plugin_hook_timeout` bounds each hook, but a hook's own timeout wins. Fixed as proposed.
- fix-proposal, LOW: the name skip also hit a root named `vendor` or a directory holding the anchor. Fixed: the scan never skips the root or a directory holding the working directory.
  New row `ClaudeFileBesideAgentsIsNamedWhenTheAnchorIsInADependencyDirectoryName`.
- fix-proposal, LOW: the effort assertion matched bare substrings. Fixed with my own change: the assertion matches the effort in backticks.
  The proposed exact wording `reasoning_effort` to the effort is not used, since F7 asks to stop pinning the wording.
- fix-affected-set: no findings.
  It left F8's contained-tests set and other docs that may describe the prompt file unchecked.

## Fixed differently

- **F7** (self-check follow-up): see the effort assertion above.

## Disputed

None.

## Deferred

None.

## Rulings

None.
