# Card 3 — prompt-suggestion-off

**Edit:**
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/session.go`

**Intent:**
Write `"promptSuggestionEnabled": false` into every run's `settings.json`:
add a `PromptSuggestionEnabled bool` field tagged `promptSuggestionEnabled` (no `omitempty`) to `settingsDoc`, left at its zero value by `buildSettings`.
A capture carries no styling, so a greyed suggestion in an empty input box reads as a draft and `IdleSession` never passes;
switching the suggestion off at the source fixes every shuttle-launched interactive session and the session-name watchdog that shares the probe.

`IdleSession` keeps its fail-closed reading of a non-empty box.
Rewrite its doc comment's first residual: the greyed suggestion no longer appears in a shuttle-launched session, and a non-empty box still classifies as not idle.
Name the setting in the `settings.go` file doc comment.

Tests: the generated settings carry `promptSuggestionEnabled` set to `false` in an interactive run, an autonomous run, and a fork run, decoded from the JSON rather than matched as a substring.

**ImpactSummary:** One new key in every generated settings file; hook content and the idle probe's logic are unchanged.
