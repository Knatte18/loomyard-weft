# session scope: skillOverrides off does not hide a plugin's skills

`plugin-skilloverrides-ignored` · Note · bug, claude, shuttle

A spawned role launched in the sandbox hub with the scope on (settings file written by claudeengine, skillOverrides naming scribe:python-testing, scribe:conversation and ten more as off) still lists all fourteen scribe skills in its skill listing. The anthropic-skills listing is gone, but that is not isolated from disableClaudeAiConnectors. The binary reads plugin skills overrides only through the synced-skill path, as the trial predicted. Evidence and numbers: docs/research/tokens/session-scope-launch-2026-11-06/report.md. A fix is another lever for plugin skills, or dropping the plugin skill names from skill_scope output.