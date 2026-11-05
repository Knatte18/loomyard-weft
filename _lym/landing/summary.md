# Claude sessions lym launches no longer credit themselves

Landing commits and pull-request bodies already end with lym's own credit block.
The Claude Code sessions lym launches got no attribution setting, though, so Claude Code's default applied: a commit a session made itself ended with `Co-Authored-By: Claude …` and a `Claude-Session:` link, and a pull request it wrote ended with "Generated with Claude Code".
That reached `main` through commits made outside a landing, such as the orch's, and showed on a pull request's own commits and comments.

## What changes

- `internal/shuttleengine/claudeengine/settings.go` adds `attributionSettings`, Claude Code's `attribution` object with `commit` and `pr` texts.
  Both settings documents carry it with no `omitempty` and leave it zero, so each renders `"attribution":{"commit":"","pr":""}`:
  - the run document `buildSettings` writes for every shuttle run;
  - the creel-only document `buildOpenSettings` writes for a session opened in a hub.
- The landing's credit block stays the one credit; no landing code changes.
- The package doc and `docs/overview.md` state the setting beside the other settings each document carries, and the open-launch paragraph no longer says the creel-only document holds hooks alone.

## Tests

- `TestSettings_CreditNoSession` pins the empty attribution in the run document, standalone and in a hub, and in the opened session's document, which had no test of its own content.
- `testdata/settings-golden.json` gains the `attribution` key in every entry, so `TestBuildSettings_MatchesGolden` keeps pinning the run document byte for byte.

## Limits

- An opened session in no hub gets no `--settings` at all, so it keeps the worktree's own settings, attribution included.
- A Codex session is outside this change: it gets no Claude settings document.
