# Card 7 — claude-user-setup-source

**Create:**
- `plan:internal/shuttleengine#UserSetupSource` -> `type UserSetupSource interface`
- `plan:internal/shuttleengine#UserSetupRequest` -> `type UserSetupRequest struct`
- `plan:internal/shuttleengine#UserSetup` -> `type UserSetup struct`
- `plan:internal/shuttleengine#InstructionItem` -> `type InstructionItem struct`
- `plan:internal/shuttleengine#OmittedHook` -> `type OmittedHook struct`
- `plan:internal/shuttleengine#HookRun` -> `type HookRun struct`
- `plan:internal/shuttleengine#HookRunner` -> `type HookRunner func(run HookRun) ([]byte, error)`
- `plan:internal/shuttleengine/claudeengine#UserSetupSource` -> `type UserSetupSource struct`
- `plan:internal/shuttleengine/claudeengine#NewUserSetupSource` -> `func NewUserSetupSource(dirs configengine.Dirs) *UserSetupSource`
- `plan:internal/shuttleengine/claudeengine#UserSetupSource.UserSetup` -> `func (s *UserSetupSource) UserSetup(req shuttleengine.UserSetupRequest) (shuttleengine.UserSetup, error)`
- `plan:internal/shuttleengine/claudeengine#instructionFiles` -> `func instructionFiles(configDir, repoRoot string) ([]shuttleengine.InstructionItem, error)`
- `plan:internal/shuttleengine/claudeengine#installedPluginRoots` -> `func installedPluginRoots(configDir, repoRoot string) (map[string]string, []string)`
- `plan:internal/shuttleengine/claudeengine#skillListing` -> `func skillListing(configDir, repoRoot, workDir string, plugins map[string]string, scope sessionScope) (string, []string)`
- `plan:internal/shuttleengine/claudeengine#pluginInstall` -> `type pluginInstall struct`
- `plan:internal/shuttleengine/claudeengine#writePluginInstalls` -> `func writePluginInstalls(t *testing.T, configDir string, installs []pluginInstall)`
- `plan:internal/shuttleengine/claudeengine#TestUserSetup_InstructionFiles` -> `func TestUserSetup_InstructionFiles(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#TestUserSetup_SkillListing` -> `func TestUserSetup_SkillListing(t *testing.T)`

**Extends:**
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `plan:internal/claudeimports#Expand`
- `internal/shuttleengine/claudeengine#claudeConfigDir`
- `internal/shuttleengine/claudeengine#resolveSessionScope`
- `internal/shuttleengine/claudeengine#readInstalledPlugins`
- `internal/shuttleengine/claudeengine#ScopeConfig`
- `internal/shuttleengine/claudeengine#LoadConfig`

**Intent:**
`claudeengine` resolves what a Claude session in a worktree loads from the user's Claude setup,
and hands it out through a `shuttleengine` seam so `codexengine` never imports `claudeengine` (D8, `PATTERN-shuttle-provider-seam`).

The seam, in a new file of `internal/shuttleengine`:
- `UserSetupSource` has one method, `UserSetup(req UserSetupRequest) (UserSetup, error)`.
- `UserSetupRequest` carries `WorkDir` (the session's working directory), `RepoRoot` (its worktree root), `Skills`, `Role`,
  `SessionID` (the lym run's id), `HookTimeout` and `RunHook HookRunner`.
- `UserSetup` carries `InstructionFiles []InstructionItem`, `SkillListing string`, `PluginContexts []InstructionItem`, `Omitted []OmittedHook` and `Warnings []string`.
- `InstructionItem` is `{Name, Text, WayForward}`; `OmittedHook` is `{Name, Reason}`.
- `HookRun` is `{Command, Dir, Env, Stdin, Timeout}`; `HookRunner` runs one and returns its stdout.
  A nil `RunHook` runs no hook, and card 8 lists each one omitted as not run.

`NewUserSetupSource(dirs)` reads claude.yaml through `dirs`; its `claudeDir` field, empty in production, names Claude Code's config directory and otherwise resolves through `claudeConfigDir`.
`UserSetup` returns, in this card:
- `InstructionFiles`: the user's global file, `CLAUDE.md` in the config directory, then each `CLAUDE.md` in a directory above `RepoRoot`, outermost first, which Claude reads walking up from its working directory.
  Each is expanded through `claudeimports.Expand`; its `Name` is its absolute path and its `WayForward` says to shorten that file.
  An absent file adds nothing.
  A read error of one of these files is the one error `UserSetup` returns, naming the file, with the way forward to fix its permissions or remove it, then launch again.
- `SkillListing`: the rule that a skill is used by reading its SKILL.md in full before following it, then one line per skill the session's scope lists, `- <name>: <description> (<absolute SKILL.md path>)`, sorted by name.
  "" when no skill is listed.

The listed skills follow the session scope `resolveSessionScope` gives claude.yaml's `session_scope`, `req.Skills` and the role:
each skill under `skills/*/SKILL.md` of a plugin the scope turns on, named `<plugin>:<skill>`;
each user skill under the config directory's `skills/*/SKILL.md`;
and each project skill under `.claude/skills/*/SKILL.md` of `WorkDir` and of each directory above it up to `RepoRoot`;
a user or project skill the scope's `SkillOverrides` turns off is left out.
With the scope disabled, every installed plugin counts as on, since lym then writes no plugin setting.
`installedPluginRoots` reads `plugins/installed_plugins.json`: for each plugin id, the `installPath` of its `user`-scope install, or of a `project`-scope install whose `projectPath` is `RepoRoot`.
A skill's description is its SKILL.md frontmatter `description:` value, empty when it has none.
As Claude's `sessionScopeFor` does with a bad plugins file, the listing never refuses a launch:
an unreadable or malformed plugins file, a plugin with no `installPath` and an unreadable SKILL.md are each left out with a text in `Warnings` naming the file.
The listing asks for no skill load; skills stay typed on the launch spec (`PATTERN-stencil-ownership`).

Test support: `writePluginInstalls` writes `plugins/installed_plugins.json` under `configDir` from `pluginInstall{ID, Scope, ProjectPath, Root}` rows, with each `installPath` a directory under the test's temp dir,
since `writeInstalledPlugins` writes a fixed `/plugins/<id>` path no plugin tree can sit at; cards 7 and 8 build plugin trees under those roots.

Tests, tier0, each building a config directory under `t.TempDir()` and setting the source's `claudeDir`:
- `TestUserSetup_InstructionFiles` (parallel table): a global file and CLAUDE.md files two and one levels above the repository root come back as the global file, then the outer, then the inner file, each expanded where it imports;
  an absent global file adds nothing; a CLAUDE.md inside the root is not listed; a global file that cannot be read returns an error naming it.
  The assertion keeps only the items under the fixture's temp base, in order, since the walk above the root also reaches the machine's own directories.
- `TestUserSetup_SkillListing` (parallel table): a scope that preloads `scribe:prose` lists both of `scribe`'s skills with their paths and descriptions and none of an off plugin's;
  a user skill named in `skills_off` and not preloaded is left out;
  a project skill under the repository root's `.claude/skills` is listed;
  a disabled scope lists every installed plugin's skills;
  a project-scope install for another repository is not listed;
  a SKILL.md with no `description:` is listed with an empty description;
  a malformed plugins file lists no plugin skill, returns no error and gives a warning naming the file;
  no skills give an empty listing.

`shuttleengine/doc.go` describes the user-setup seam; `claudeengine/doc.go` describes what the source resolves and from where.

**ImpactSummary:** adds a new seam file in `shuttleengine` and a new source in `claudeengine`; no existing symbol changes, and nothing calls the source before card 9.
