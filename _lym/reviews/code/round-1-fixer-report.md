# Fixer report — code review round 1

Every finding F1 to F9 is fixed. `lym gate test ./internal/shuttleengine/codexengine` passes at tier 0 (94 passed, none failed).
F1 to F5 and F9 are fixed as proposed, F6 is fixed as proposed, F7 and the self-check's test finding are fixed with own changes.
The commits group files, not findings, since F5 to F9 share `skillload.go`: `0cb73c0e` (F1), `60d1e95c` (F3), `598d59ca` (F2, F4), `1cea8589` (F5 to F9), and a last commit for the self-check.

## Gate repair

The gate found three comment blocks added this round wrapped mid-sentence (`guards_test.go` on `TestGuardEntries_RenderEveryRowWithItsCodexText`, `skillload_test.go` on `TestClassifySkillLoad` and `TestClassifySkillLoad_NewerVersionAfterTheLine`).
Each is now one sentence per line, with no change to what it says; a further commit holds the repair.

## Changes

- **F1** — `docs/overview.md`: the shuttle entry now says the `codex` engine loads skills as SKILL.md files the model reads by absolute path under `skill_roots`, and installs the shell guard rows as `PreToolUse` hooks matching `Bash` with Codex's own steer and notice texts; fork audit and creel hooks stay listed as missing.
  The self-check's wording finding made the sentence name the switches: python and publish rows follow their switches, and the refuse-raw row needs a non-empty `refuse_raw` list.
- **F2, F4** — `prepare_test.go`: new `TestPrepare_InstallsGuardsFromItsInputs`, one table with a python-switch-only row, a publish-guard-only row, a seeded `refuse_raw` row and a gate config that does not load.
  Each success row compares the launch line and the prompt file against `installedGuards` with the matching inputs; the failing row asserts an error beginning `codexengine:` and no `prompt.md`.
- **F3** — `guards_test.go`: `TestGuardEntries_RenderEveryRowWithItsCodexText` asserts the literal deny output and that every entry opens with the literal `{matcher="Bash",hooks=[`; the rebuild of the entry from `hookEntry` is dropped.
  Fixed differently in one respect: the self-check showed that dropping it left no tie between an entry and its row's command, so I kept a weaker assertion that the entry contains `tomlString(guardCommand(row))`.
- **F5** — `skillload_test.go`: rows for `custom_tool_call` and `local_shell_call` payloads, a repeated load message, and a truncated line; `TestSkillLoadMessage_ExpandsAHomeRoot` (not parallel, says so) and `TestSkillLoadMessage_WarnsOnceWhenCodexYamlDoesNotLoad`; version rows in `TestSkillPath` (pre-release, non-numeric folder, the `2`, `10`, `1a` cycle).
  The rollout line builders moved out of the test into package helpers.
- **F6** — `skillload.go`: `compareVersions` orders fields by one total rule (numbers by value, a number above text, text by bytes) through `compareVersionFields`, documented in the function comment; a field with a numeric prefix and a suffix is text because the parse is `ParseUint`.
- **F7** — fixed with my own change, not the proposed escaping: `skillPath` lists marketplace and version folders with `os.ReadDir` (`skillFiles`) instead of `filepath.Glob`, so no pattern exists to escape.
  The proposed backslash escaping does not work on Windows, where the backslash is the path separator and `filepath.Glob` has no escape; the walk resolves a root holding `[` on every platform. `TestSkillPath` has a `root[1]` root row.
- **F8** — `skillload.go`: `skillLoadOpening` and `skillLoadNone` are constants shared by `skillLoadLine` and `isSkillLoadMessage`; `loadTurnCalls` matches a message by that fixed text, so a version that appeared after the launch no longer hides the line. `TestClassifySkillLoad_NewerVersionAfterTheLine` covers it.
- **F9** — `skillload.go` and `codexengine.go`: `skillLoadOver` builds the line over already expanded roots and `expandSkillRoots` expands a held `Config`; `Prepare` calls them with the `codex` it already loaded, so one load serves the launch.

## Disputed

- **Self-check, fix-proposal finding 3** (premise false): the finding says a load line that named no file now classifies as verified with the later-resolvable skill Missing, and calls that wrong.
  `run.go:906-915` takes `Missing` from a verified report and returns it for a retry, which sends a fresh line naming the file; the old unverified result (`run.go:907-910`) skipped that retry, so Missing is the remedy.
- **Self-check, fix-proposal finding 4** (every fix worse): a digit-only hash folder such as `123456789012` parses as a number in field 0 and outranks `1.10.0`.
  Every rule that separates it from a version needs a threshold or a shape test, such as a digit count or a dot requirement, which would misorder a real version like `2026` or `20261103` and is a rule no requirement asks for; the numbers-above-text rule is the one F6 prescribes.

## Deferred

Nothing deferred.

## Self-check

- fix-code-correctness — no findings.
- fix-consistency — no findings.
- fix-proposal — finding 1, LOW, the overview overstates which guard rows install: fixed as proposed (overview wording names the switches and the refuse list).
- fix-proposal — finding 2, LOW, no test ties an entry to its row's command: fixed as proposed (the contains assertion over `tomlString(guardCommand(row))`).
- fix-proposal — finding 3, LOW, none-line classified verified after a root gains a skill: disputed, premise false.
- fix-proposal — finding 4, LOW, a digit-only hash folder outranks a version: disputed, every fix worse.
- fix-affected-set — no findings.
