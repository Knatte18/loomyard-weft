# Card 7 — claude-user-setup-source

**Create:**
- `internal/shuttleengine#UserSetupSource`
- `internal/shuttleengine#UserSetupRequest`
- `internal/shuttleengine#UserSetup`
- `internal/shuttleengine#InstructionItem`
- `internal/shuttleengine#OmittedHook`
- `internal/shuttleengine#HookRun`
- `internal/shuttleengine#HookRunner`
- `internal/shuttleengine/claudeengine#UserSetupSource`
- `internal/shuttleengine/claudeengine#NewUserSetupSource`
- `internal/shuttleengine/claudeengine#UserSetupSource.UserSetup`
- `internal/shuttleengine/claudeengine#instructionFiles`
- `internal/shuttleengine/claudeengine#installedPluginRoots`
- `internal/shuttleengine/claudeengine#skillListing`
- `internal/shuttleengine/claudeengine#pluginInstall`
- `internal/shuttleengine/claudeengine#writePluginInstalls`
- `internal/shuttleengine/claudeengine#TestUserSetup_InstructionFiles`
- `internal/shuttleengine/claudeengine#TestUserSetup_SkillListing`

**Extends:**
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/claudeimports#Expand`
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
When `claudeConfigDir` fails, `UserSetup` leaves out only the parts that need the config directory:
the user's global file, the user skills and plugin skills of the listing, and card 8's plugin session-start contexts.
It keeps the CLAUDE.md files above `RepoRoot` and the project skills, adds one `Warnings` entry naming the cause, with the way forward to set `CLAUDE_CONFIG_DIR` or `HOME`, and never returns an error,
as `sessionScopeFor` does with an unreadable plugins file.
`UserSetup` returns, in this card:
- `InstructionFiles`: the user's global file, `CLAUDE.md` in the config directory, then each `CLAUDE.md` in a directory above `RepoRoot`, outermost first, which Claude reads walking up from its working directory.
  Each is expanded through `claudeimports.Expand`; its `Name` is its absolute path and its `WayForward` says to shorten that file.
  An absent file adds nothing.
  A read error of the global file is the one error `UserSetup` returns, naming the file, with the way forward to fix its permissions or remove it, then launch again.
  An unreadable CLAUDE.md above `RepoRoot` is left out of `InstructionFiles` with a `Warnings` entry naming it, since the walk reaches machine directories the user may not own.
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
  an absent global file adds nothing; a CLAUDE.md inside the root is not listed; a global file that cannot be read returns an error naming it;
  an unreadable CLAUDE.md above the root is not listed, returns no error and gives a warning naming it.
  Both unreadable rows make the file unreadable with a directory standing at its path:
  `os.ReadFile` of a directory fails with an error other than `fs.ErrNotExist` for every user, root included,
  so neither row skips itself under root, and each tells an unreadable file from an absent one.
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
