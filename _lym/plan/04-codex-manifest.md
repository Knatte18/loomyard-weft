# Card 4 — Codex's Prepare writes its manifest

**Edit:**
- `internal/shuttleengine/codexengine#Codex.Prepare`
- `internal/shuttleengine/codexengine#Codex.resolveUserSetup`
- `internal/shuttleengine/codexengine/doc.go`
- `internal/shuttleengine/codexengine#TestPrepare_UserSetup`

**Create:**
- `internal/shuttleengine/codexengine#codexManifest`
- `internal/shuttleengine/codexengine#manifestInputs`
- `internal/shuttleengine/codexengine#TestPrepare_WritesTheManifest`

**Uses:**
- `internal/shuttleengine#WriteManifest`
- `internal/shuttleengine#ReadManifest`
- `internal/shuttleengine#Manifest`
- `internal/shuttleengine#ErrHookUnrun`
- `internal/shuttleengine/codexengine#newRepoFixture`
- `internal/shuttleengine/codexengine#repoFixture.addWorktree`
- `internal/shuttleengine/codexengine#newEngine`
- `internal/shuttleengine/codexengine#fakeUserSetupSource`
- `internal/shuttleengine/codexengine#claudeFileIsUnread`
- `internal/heddleengine#Marker`

**Intent:**
`Codex.Prepare` writes the D1 manifest into the run directory through `shuttleengine.WriteManifest`, after `prompt.md` and the batching state are written and before it returns the launch.

D1 reads the manifest back from the launch.
`manifestInputs` carries each candidate item with the exact text or hook command its delivery puts on the launch line, and the launch line `buildLaunchCmd` returned.
`codexManifest` keeps a candidate only when the line holds its text or command, in the TOML string form `developerInstructionsSetting` and `hookSettings` write, so a delivery dropped anywhere in `Prepare` drops its item:

- `prompt-engine`, as Claude's.
- `standing-passage`, one per standing passage the developer instructions hold, key the slug, `Engine` the passage's own.
- `instruction-source`: each user-setup instruction file the developer instructions hold, key its absolute path;
  each CLAUDE.md on the path from the checkout root to the working directory, key its root-relative path, delivered when Codex reads it itself or the instruction-file pointers name it (`claudeFileIsUnread` decides which);
  each `Standing.FileEntries` entry, key `heddle-file/<slug>`, `Engine` the entry's own, delivered only when the working directory's `AGENTS.override.md` opens with `heddleengine.Marker` and holds the entry's text.
- `skill-load`, one per skill whose SKILL.md the load line names; a skill no root holds has no item, so it shows missing against Claude.
- `skill-listing`, one per `ListedSkills` name when the developer instructions hold the listing.
- `plugin-context`: each `PluginContexts` item the developer instructions hold, delivered; each `Omitted` hook with a `Plugin`, omitted with its reason; key `<plugin>/<hook command>`.
- `guard`: `question-deny` when the question deny hook is on the line, and each installed shellguard row by name.
- `batching-nudge` when its PostToolUse hook is on the line.
- `notice`: `deny-notice` and `creel-trust` from the creel head item, `subagent-passage` when that item is non-empty, each when the developer instructions hold it.
- `settings`, keyed to `codex`, one item for the sandbox mode in `Detail` and one for the developer-instructions setting's byte size.

`resolveUserSetup` logs no per-hook warning for an omitted hook whose reason is `shuttleengine.ErrHookUnrun`'s text, so a hook listed unrun is not reported as a hook to fix; every other omission still warns with its way forward.
A write failure refuses the launch with the path.

Tests, tier0:
`TestPrepare_WritesTheManifest` is a table on `newRepoFixture`, `addWorktree` and `newEngine`, with `fakeUserSetupSource` giving one instruction file, two listed skills, one delivered and one omitted plugin hook, and an installed passage reader giving one passage and one file entry.
Each row reads the record back with `ReadManifest` and asserts its items by channel and key:
- an autonomous spec with skills holds `prompt-engine`, `question-deny`, every unswitched guard row, a `skill-load` item for a skill a root holds and none for a skill no root holds, the `deny-notice`, `creel-trust` and `subagent-passage` notices, the instruction file, both listed skills, and the passage by slug with its engine;
- an interactive spec holds no `question-deny`;
- a spec with `DenyPublish` and a config with `DenyPython` hold their switched guard rows;
- a batching-nudge-on config holds `batching-nudge` and an off one holds none;
- a CLAUDE.md Codex reads itself and one only a pointer names each give a delivered `instruction-source` item;
- the file entry appears as `heddle-file/<slug>` under a marked `AGENTS.override.md` and has no item under an unmarked one;
- the delivered and the omitted plugin hook give a delivered and an omitted `plugin-context` item, and the record's `Verdict` names the omitted one;
- the `settings` items carry `codex` and the sandbox mode.

`TestPrepare_UserSetup` gains a row beside `OmittedHookAndWarningReachTheWarnings`: an omitted hook whose reason is `shuttleengine.ErrHookUnrun`'s text adds nothing to the `warnings` buffer.
That existing row already pins the warning for any other reason.

**ImpactSummary:** `Codex.Prepare` writes one more file read back from its launch line; `resolveUserSetup` drops one warning for unrun hooks; no signature changes.

**Commit:** 4: Codex's Prepare writes its launch manifest
