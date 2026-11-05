## Goal

```markdown
## Seen

In the scribe hub (2026-11-05) the orch failed `Skill(ly:board)` with "Unknown skill", and shuttle logged `skill skipped ... ly:board cause=unknown`.
The user settings set `ly@loomyard: false`; only loomyard's own checked-in `.claude/settings.json` turns it on, so every other hub (quarry, scribe, lyt, cs) starts its orch and agents without the ly plugin.
Meanwhile every session, in every hub, loads plugins, skills and connectors it never uses: report `docs/research/tokens/startup-context-2026-11-05/report.md` puts that at about 11,000 tokens per spawned role and 7,900 per orch start.

## Decision (operator, 2026-11-05)

Plugins and skills are available only where they are needed, scoped down hard.
lym, not the hub's repository, decides what each session gets, through the settings file shuttle already writes and passes with `--settings` (`internal/shuttleengine/claudeengine`); no file is added to any hub's repository.

## Change

- Each role gets an explicit allowlist of plugins and skills; everything else is off for that session, whatever the user settings or the repository's `.claude/settings.json` say.
  - `ly@loomyard` is on for the orch and the board session in every hub, and off for roles that do not use its skills (writers, reviewers, fixers, Webster, drivers, unless one does).
  - scribe skills: only those the role's stencil and `language.yaml` preload; the other languages' skills are off.
  - `prowler` off unless a role uses it.
- Off for every session: claude.ai connectors (`disableClaudeAiConnectors`), the `anthropic-skills` listing (`skillOverrides`), bundled skills (`disableBundledSkills`, the orch may keep `update-config` only if needed).
- Auto-memory off for spawned roles (`autoMemoryEnabled: false`); the orch keeps it.
- `--disallowedTools ListAgents` for spawned roles.
- Verify in a launch that each lever works as the report's table says (none was tried there), and measure one spawned role's and the orch's start load after the change.
- A test checks the written settings per role.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-05): Launch check and start-load measurement move to a docs task

Decision: The run lands the per-role settings and their tests only. Checking in a live launch that each lever works, and measuring the start load before and after, moves to note session-scope-launch-trial, a darn docs task released after this lands.

Rationale: A run cannot launch Claude headless, and trials run as darn docs tasks.

### Added after Discussion (parent, 2026-11-05): Interactive sessions are scoped too; only the orch keeps memory and ListAgents

Decision: Refuse the blanket interactive exception. Auto-memory stays on and ListAgents stays allowed for the orch alone: PR-Review, the sampler, the board session and every other session lym launches get memory off and ListAgents denied. Plugin skills follow one rule in every session, interactive or not: the ly plugin keeps all its skills where it is on, since the operator types them as slash commands, and every other plugin is limited to the skills the session preloads. So the orch keeps only its preloaded scribe skills.

Rationale: The operator asked for plugins and skills to be scoped down hard, available only where needed. An interactive session does not need other languages' skills or the orch's memory index; ly's slash commands are the one case an operator types by hand.

### Added after Discussion (parent, 2026-11-05): ListAgents is denied through permissions.deny in the settings

Decision: Accept permissions.deny in the written settings in place of --disallowedTools on the launch line.

Rationale: The behavior is the one asked for, one settings document carries the whole scope, and the resume line needs no second flag.

### Added after Discussion (parent, 2026-11-05): The board button preloads scribe:prose and scribe:conversation

Decision: Accept the preload in both reed templates and config_test.go.

Rationale: The board session talks with the operator, so it needs those two rules, and with plugins following preloads it would otherwise lose them.

### Added after Discussion (parent, 2026-11-05): A session opened outside a hub gets no scope

Decision: Accept the narrowing: the scope covers sessions lym launches in a hub. An opened session outside a hub writes no settings, as today.

Rationale: Outside a hub there is no hub config or settings location to write to, and the operator's concern is the hubs' sessions.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - board-button-scribe-preload-unlicensed (class decision, severity MEDIUM, open in rounds 1)
  - interactive-sessions-keep-memory-listagents-plugin-skills (class decision, severity MEDIUM, open in rounds 1)
  - listagents-denied-in-settings-not-launch-line (class decision, severity MEDIUM, open in rounds 1)
  - no-scope-for-opened-session-outside-hub (class decision, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
