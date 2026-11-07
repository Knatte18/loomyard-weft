```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

- Shipped `lym-session-scope` and `lym-prompt-suggestion-off` carry the right keys.
  - Pinned by `settings_test.go:277-279`, a golden of the shipped object. Slug and order are also pinned in `store_test.go:188`.
- Derived `lym-agent-defaults` (`effortLevel` plus `env.CLAUDE_CODE_SUBAGENT_EFFORT`).
  - Pinned by `TestClaudeSettingsCarryTheAgentDefaults`, with rows for template default, hub default and unloadable file.
  - It checks Show and Render, and the warning.
  - The `derivedSettings` constants in `TestRenderClaudeSettings` pin the merge with other entries.
- `fileText` codex and claude-file filter, now `Layer == LayerRepository`, with derived and shipped going to the shipped section.
  - None. The derived entry targets only `claude-settings`, so no test can reach the changed branch. The behaviour is unchanged for shipped entries and is covered by the existing codex render tests.
- `Layers.Ordered` places derived entries after shipped and before repository.
  - Partly. The derived-before-repository precedence is not asserted on a conflicting key. `derivedBesideEnvA` only shows a non-conflicting merge.
- `RenderOverview` sections, source lines, fenced blocks, "No entry.", idempotence.
  - Pinned by `TestRenderOverview`, which also checks the `changed` return.
- Overridden shipped entry's source line and text.
  - Pinned by `TestRenderOverviewNamesAnOverriddenShippedEntry`.
- `RenderOverview` when `agents.yaml` is unloadable (derived entry left out silently).
  - None. `overview_test.go` has no invalid-agents row.
- `RenderOverview` read and write error paths (`read HEDDLE.md`, `write HEDDLE.md`).
  - None.
- `entryBlock` fence widening past 3 backticks, and a JSON-object entry whose text is not an object.
  - None. Only a plain markdown block and a valid object are asserted.
- `targetReach` text, including the codex line.
  - None, and it is static prose.
- `Replayer.Claims` includes `HEDDLE.md`.
  - Pinned by `NameAndClaims`.
- `Replayer.Apply` renders the overview and sets `Empty` from `!applied && !rendered`.
  - Pinned by the two-pass `CommitWithNoEntryChange...` test (not empty, then empty) and by `AppliedChangeRendersTheOverviewFromWhatOriginHolds`.
- Replayer render failure returns `render HEDDLE.md: ...`.
  - None.
- Replay of a commit carrying a conflicting `HEDDLE.md`: the commit's own overview never lands.
  - None. `replayer_test.go` requests only pass `FileName`, so the `HEDDLE.md` claim is never driven with a `HEDDLE.md` file. `replay_integration_test.go` has the `originHeddleCommit` helper, but I did not see the local-conflict case it serves.
- `lym heddle set` and `remove` render the overview (`saveLocked`).
  - `set` is pinned by `cli_integration_test.go:230-238`.
  - `remove`, override `set`/`remove` and `ChangeStore` (the `lym orch rules` path) are not asserted.
  - All of these go through `saveLocked`, so one path covers them. This is not a gap.
- `saveLocked` render-failure message ("is saved but ... was not rendered").
  - None.
- `claudeengine` documents drop `attribution`, `promptSuggestionEnabled`, `disable*` and `permissions`.
  - Pinned by `TestBuildSettings_LeavesWorktreeSettingsOut`, over every run mode in and out of a hub. It also covers `scopeSettingsKeys` in the open-launch doc.
- `ListAgents` deny removed for non-orch roles.
  - Pinned by `scope_test.go` (`want` strings without `permissions`, and the `doc["permissions"]` absence check).
- Orch keeps auto-memory and non-orch has it off.
  - Pinned by `TestSessionScope_WrittenSettingsPerRole` and `...ReachesWrittenSettings`.
- `agentconfig.SubagentEffortEnv` constant used by `claudeengine`.
  - Pinned by `subagents_test.go:126`.
- Per-role `--effort` stays on the launch, with no `command.go` logic change.
  - Existing tests; nothing new is needed.
- `claudeengine` golden `settings-golden.json`.
  - Updated in the diff. I could not read its content, so I did not judge it.

**Findings**

1. MEDIUM, `design`. `internal/heddleengine/overview_test.go`, `replayer.go:108-111`, `overview.go:36-37`.
   - Behaviour: the overview drops the derived entry silently when `agents.yaml` does not load, and `RenderOverview` ignores `DerivedWarning`.
   - Coverage: no row. Only the worktree render has an invalid-agents row (`invalidAgentsConfig` in `settings_test.go`).
   - Fix: add an invalid-`agents.yaml` row to `TestRenderOverview`, asserting the section holds no derived entry and the render still succeeds.

2. MEDIUM, `design`. `internal/heddleengine/replayer_test.go`.
   - Behaviour: the claim on `HEDDLE.md` means a commit's own overview is discarded and re-rendered from origin's state, so two hubs never conflict (`replayer.go:61`, commit message).
   - Coverage: no test drives `Apply` with a `HEDDLE.md` file in `req.Files`, or with a commit whose overview differs from what origin renders.
   - Fix: add a replayer row that passes a stale `HEDDLE.md` in the request and asserts the file on disk holds the origin-derived text.

3. LOW, `design`. `internal/heddleengine/overview_test.go`, `overview.go:62-72`.
   - Behaviour: fence widening for text that holds backtick runs, and the pass-through of a JSON-object entry whose text is not an object.
   - Coverage: none. Both branches are untested.
   - Fix: add one row each to `TestRenderOverview`.

4. LOW, `design`. `internal/heddleengine/settings_test.go`.
   - Behaviour: a repository entry outranks the derived entry on a conflicting key, because `Ordered` puts derived before repository.
   - Coverage: no row sets `effortLevel` or `env.CLAUDE_CODE_SUBAGENT_EFFORT` in a repository entry.
   - Fix: add that row to `TestRenderClaudeSettings`.

5. LOW, `design`. `internal/heddlecli/cli_integration_test.go:230`.
   - Behaviour: `saveLocked` fails with "saved but not rendered" when the overview cannot be written.
   - Coverage: none for that error path.
   - Fix: add a row only if a seam makes it cheap, since the cli tests already use seams. Otherwise accept the gap.

**Tool calls**
1. Bash: `settings_test.go` diff row names and warning rows, which the manifest did not show.
2. Bash: `claudeengine/settings_test.go` diff and the replay integration diff, which the manifest did not show.
3. Bash: grep for the shipped-key assertions, overview references and `DerivedWarning` coverage across the tests.
