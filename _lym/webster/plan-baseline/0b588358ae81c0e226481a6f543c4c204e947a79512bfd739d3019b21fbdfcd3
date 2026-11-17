# Card 6 — codex-subagent-terms

**Create:**
- `internal/shuttleengine/codexengine#Codex.subagentEffort`
- `internal/shuttleengine/codexengine#subagentPassage`
- `internal/shuttleengine/codexengine#TestPrepare_SubagentTerms`
- `internal/shuttleengine/codexengine#TestLoadConfig_RefusesInvalidValues`

**Edit:**
- `internal/shuttleengine/codexengine#Config`
- `internal/shuttleengine/codexengine#ConfigSpec`
- `internal/shuttleengine/codexengine#LoadConfig`
- `internal/shuttleengine/codexengine#launchLine`
- `internal/shuttleengine/codexengine#codexCommand`
- `internal/shuttleengine/codexengine#Codex.Prepare`
- `internal/shuttleengine/codexengine#TestBuildLaunchCmd`
- `internal/shuttleengine/codexengine#TestLoadConfig_Template`
- `internal/shuttleengine/codexengine#TestPrepare_Launch`
- `internal/shuttleengine/codexengine#TestPrepare_SandboxMode`
- `internal/shuttleengine/codexengine#TestPrepare_ExpandsHomeInWritableRoots`
- `internal/shuttleengine/codexengine#TestPrepare_InstallsGuardsFromItsInputs`

**Extends:**
- `internal/shuttleengine/codexengine/template.yaml`
- `internal/shuttleengine/codexengine/doc.go`

**Uses:**
- `internal/agentconfig#Load`
- `internal/shuttleengine/codexengine#developerInstructions`
- `internal/shuttleengine/codexengine#tomlString`
- `internal/shuttleengine/codexengine#repoFixture.addWorktree`
- `internal/shuttleengine/codexengine#repoFixture.writeCodexLayer`

**Intent:**
A Codex session gives its subagents Claude's subagent terms, in Codex's words (D6).

codex.yaml gains `max_subagent_threads`, template default 16, as `Config.MaxSubagentThreads`, classified `CategoryNone` in `ConfigSpec`;
`LoadConfig` refuses a value below 1 as invalid, naming the key, and `loadConfigOrRefuse` adds its way forward.
The template comment says it raises Codex's default cap of three concurrent children per session.

`subagentEffort` returns the effort agents.yaml gives the role on a model it does not list, the second result of `agents.ModelEfforts(role)`,
since a Codex model is none of the Claude models agents.yaml keys efforts by; that is the counterpart of Claude's `subagentEffortEnv`.
An effort codex.yaml's `efforts` does not list returns "" and logs a warning through `c.warn` naming the effort and `lym config codex`, so the session's subagents take Codex's default rather than a value Codex rejects.

`launchLine` gains `subagentEffort` and `maxSubagentThreads`.
`codexCommand` adds `-c agents.default_subagent_reasoning_effort=<toml string>` when the effort is set,
and `-c agents.max_concurrent_threads_per_session=<n>` when the cap is above zero, both after the effort setting and before the hook settings.
`Prepare` sets both from `cfg.Agents`, `spec.ConfigRole` and codex.yaml; `OpenLaunch` leaves both zero, so an opened line is unchanged and carries no cap.

`subagentPassage` returns "" for an empty effort.
Otherwise it says, in Codex's terms and naming no Claude tool:
every fresh `spawn_agent` call sets `reasoning_effort` to that effort and leaves `model` unset, so the child runs on the session's model;
a `fork_turns: "all"` fork shares the session's effort and sets none;
and, when the spec has skills, the brief names those of the session's skills that fit the child's work, for it to load by reading each skill's SKILL.md before anything else, listing the skills.
`Prepare` inserts it as the developer-instructions item named `the subagent passage` after the creel head, where Claude's appended prompt holds its subagent passage.

Tests:
- `TestPrepare_SubagentTerms` is a tier0 parallel table over `cfg.Agents` loaded with `agentconfig.Load` from an agents.yaml written for the fixture worktree.
  Each row builds its want line through `developerInstructions` and `tomlString`, never by searching the quoted setting for raw text.
  Rows: an effort codex.yaml lists gives the agents key and the passage naming that effort and the skills;
  the same with no skills gives a passage naming no skill;
  an effort codex.yaml does not list (`xhigh`) gives neither, and a warning naming it;
  no effort for the role gives neither;
  `ConfiguredThreadCap` writes `max_subagent_threads: 24` with `writeCodexLayer` and expects `agents.max_concurrent_threads_per_session=24`.
  Every other row expects `=16`.
- `TestLoadConfig_RefusesInvalidValues` is a tier0 table over a written local layer, each row naming the key the refusal carries; its first row is `max_subagent_threads: 0`.
- `TestLoadConfig_Template` expects `MaxSubagentThreads: 16`.
- `TestBuildLaunchCmd`'s `EveryInput` row sets both fields and its want carries both settings; the other rows set neither.
- The four full-line Prepare tests add `maxSubagentThreads: 16` to their want line.

`codexengine/doc.go` gains a paragraph on the agents-table settings and the subagent passage.

**ImpactSummary:** codex.yaml gains one key with a classified category; every Prepare launch line gains the thread cap and, with a role effort, the subagent effort and passage; OpenLaunch and Claude are untouched.
