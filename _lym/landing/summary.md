# Sessions lym launches add no Claude co-author or session link
Claude Code sessions that lym launches no longer credit themselves in commits or pull requests.
Before, they got no attribution setting, so Claude Code's default applied.
Every commit a session made itself ended with `Co-Authored-By: Claude …` and a `Claude-Session:` link, and a pull request it wrote ended with "Generated with Claude Code".
That covered samplers, PR-Review fixers, writers, fixers and the orch, and it reached `main` through commits made outside a landing.
The landing's own credit block is now the one credit.

## What changes

- `internal/shuttleengine/claudeengine/settings.go`: both settings documents the engine composes set Claude Code's `attribution` to `{"commit": "", "pr": ""}`.
  These are the run document and the creel-only document written for an opened session.
- The new `attributionSettings` type has no `omitempty` and the builders leave it zero, so the empty texts are always written.
- `testdata/settings-golden.json` and the settings and command tests pin the new key in both documents.
- The package doc, the `OpenLaunch` and `writeOpenFiles` comments, and `docs/overview.md` state the attribution beside the other settings each document carries.

## Design decisions

- The empty string is the form Claude Code documents for turning attribution off, so no session-side flag or environment variable is involved.
- The opened-session document otherwise stays hooks-only, so the worktree's own settings still govern everything else.

## To check by hand

- Launch a session with `lym shuttle open` in a hub and with a normal run, make a commit in each, and confirm the message carries no `Co-Authored-By` or `Claude-Session:` line.
  Neither was run here.
- The `OpenLaunch` comment in `command.go` was reworded to name the attribution beside the hook delivery; check that no other comment still says the opened-session document holds hooks alone.
