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

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
