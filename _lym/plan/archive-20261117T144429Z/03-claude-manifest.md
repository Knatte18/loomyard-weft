# Card 3 — Claude's Prepare writes its manifest

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#UserSetupSource.UserSetup`
- `internal/shuttleengine/claudeengine#skillListing` -> `func skillListing(configDir, repoRoot, workDir string, plugins map[string]string, scope sessionScope) (string, []string, []string)`
- `internal/shuttleengine/claudeengine#pluginSessionStartContexts`
- `internal/shuttleengine/claudeengine#TestUserSetup_SkillListing`
- `internal/shuttleengine/claudeengine#TestUserSetup_PluginSessionStartContexts`
- `internal/shuttleengine/claudeengine/doc.go`

**Create:**
- `plan:internal/shuttleengine/claudeengine#claudeManifest` -> `func claudeManifest(in manifestInputs) shuttleengine.Manifest`
- `plan:internal/shuttleengine/claudeengine#manifestInputs` -> `type manifestInputs struct`
- `plan:internal/shuttleengine/claudeengine#TestPrepare_WritesTheManifest` -> `func TestPrepare_WritesTheManifest(t *testing.T)`

**Uses:**
- `plan:internal/shuttleengine#WriteManifest`
- `plan:internal/shuttleengine#ReadManifest`
- `plan:internal/shuttleengine#Manifest`
- `internal/shuttleengine/claudeengine#engineWithAnchor`
- `internal/shuttleengine/claudeengine#instructionFiles`
- `internal/shuttleengine/claudeengine#buildDenyNotice`
- `internal/shuttleengine/claudeengine#appendPromptText`
- `internal/heddleengine#Marker`

**Intent:**
`Claude.Prepare` writes the D1 manifest into the run directory through `shuttleengine.WriteManifest`, after it has written `prompt.md`, `settings.json` and the appended system prompt, and before it returns the launch.

D1 reads the manifest back from the launch.
`manifestInputs` carries each candidate item with the exact text or hook command its delivery puts into an artifact, and the artifacts as written: the bytes of `settings.json` and of the appended system prompt read back from the run directory, and the launch line `buildLaunchCmd` returned.
`claudeManifest` keeps a candidate only when that artifact holds its text or command, so a delivery dropped anywhere in `Prepare`, an argument of `buildSettings` or `appendPromptText` included, drops its item:

- `prompt-engine`, key `prompt-engine`, delivered when the spec passed `CheckPromptEngine` with a prompt engine (a hand-written prompt gives key `hand-written`).
- `standing-passage`, one per `Standing.Passages` entry the appended prompt holds, key the slug, `Engine` the passage's own.
- `instruction-source`, each native:
  the user's global CLAUDE.md and each CLAUDE.md above the repository root, from the same `instructionFiles` resolution the user-setup source uses, key the absolute path;
  each CLAUDE.md on the path from the repository root to the working directory, key its root-relative path;
  each `Standing.FileEntries` entry, key `heddle-file/<slug>`, `Engine` the entry's own, delivered only when the anchor's `CLAUDE.local.md` opens with `heddleengine.Marker` and holds the entry's text.
  The working directory is the engine's anchor resolved with `filepath.EvalSymlinks`, as Codex resolves it, and the repository root is the nearest directory at or above it holding a `.git` entry, so both engines key the same files the same way.
- `skill-load`, one per spec skill the launch line's load message names, key the skill name.
- `skill-listing`, native, one per `ListedSkills` name of the engine's own `UserSetupSource` resolution for the spec's skills and role.
- `plugin-context`, native, one per session-start hook of a plugin the scope turns on, key `<plugin>/<hook command>`; the resolution runs with no hook runner, so no hook runs.
- `guard`, one per PreToolUse deny the written `settings.json` carries, key the shellguard row name, `question-deny` for the AskUserQuestion deny and `webster-fork-deny` for the `lym webster` fork deny.
- `batching-nudge`, key `batching-nudge`, when the written settings carry its PostToolUse hook.
- `notice`, keys `deny-notice` (when the notice is non-empty), `creel-trust` (always) and `subagent-passage` (when non-empty), each when the appended prompt holds it.
- `settings`, keyed to `claude`, one item naming the settings document with its byte size in `Detail`.

`skillListing` also returns the listed names in listing order, and `UserSetupSource.UserSetup` sets `ListedSkills` from them;
`pluginSessionStartContexts` sets `Plugin` and `Hook` on each item and each omitted hook it reports for a hook (a hooks file it cannot read keeps them empty).
A write failure of the manifest refuses the launch with the path, as the other artifact writes do.

Tests, all tier0:
`TestPrepare_WritesTheManifest` is a table on `engineWithAnchor` whose rows set the engine's `claudeDir` to a fixture config directory holding a global CLAUDE.md, an installed plugin with one session-start hook and a user skill, and write a CLAUDE.md at the anchor and a marked `CLAUDE.local.md`.
Each row reads the record back with `ReadManifest` and asserts its items by channel and key:
- an autonomous spec with skills holds `prompt-engine`, the question deny, every unswitched guard row, each skill load, the `deny-notice`, `creel-trust` and `subagent-passage` notices, the native global CLAUDE.md and anchor CLAUDE.md sources, the listed skills and the plugin's context keyed `<plugin>/<hook command>`;
- a hand-written prompt holds `prompt-engine` under key `hand-written`;
- an interactive spec holds no `question-deny`;
- a spec with `ForkSubagents` holds `webster-fork-deny` with an empty `Engine`;
- a spec with `DenyPublish` and a config with `DenyPython` hold their switched guard rows, and a spec with neither holds none of them;
- a batching-nudge-on config holds `batching-nudge` and an off one holds none;
- an installed reader's passage appears by slug with its engine, and its file entry appears as `heddle-file/<slug>`;
  with an unmarked `CLAUDE.local.md` the file entry has no item;
- the `settings` item carries `claude`.
`TestUserSetup_SkillListing` asserts `ListedSkills`, and `TestUserSetup_PluginSessionStartContexts` asserts `Plugin` and `Hook` on a context and an omitted hook.

**ImpactSummary:** `Claude.Prepare` writes one more file read back from its own artifacts; `skillListing` gains a result with one caller in `UserSetupSource.UserSetup`; the Codex engine sees the new user-setup fields filled.

**Commit:** 3: Claude's Prepare writes its launch manifest
