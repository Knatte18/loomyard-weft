# Sessions in hubs other than loomyard lack the ly skills

`ly-plugin-off-in-other-hubs` · Note · bug, shuttle

## Seen

In the scribe hub (2026-11-05) the orch failed `Skill(ly:board)` with "Unknown skill", and shuttle logged `skill skipped ... ly:board cause=unknown`. The user settings set `ly@loomyard: false`; only loomyard's own checked-in `.claude/settings.json` turns it on, so every other hub (quarry, scribe, lyt, cs) starts its orch and agents without the ly plugin.

## Expected

lym turns the plugin on for every session it launches, in any hub, through the settings file shuttle already writes and passes with `--settings` (`internal/shuttleengine/claudeengine`), e.g. `"enabledPlugins": {"ly@loomyard": true}`, without adding any file to the hub's repository. A test checks the written settings carry it.