# Sessions lym launches add no Claude co-author or session link to commits and PRs

`sessions-no-self-credit` · Task · bug, claude, shuttle · running · Code-Burler-Review

## Seen (2026-11-05)

Landing commits and pull-request bodies end with lym's own credit block (`364c47ca`), and the operator does not want an engine credited as a co-author.
But the Claude Code sessions lym launches get no attribution setting, so Claude Code's default applies: every commit a session makes itself ends with `Co-Authored-By: Claude …` and a `Claude-Session:` link, and a pull request it writes ends with "Generated with Claude Code".
That covers samplers, PR-Review fixers, writers, fixers and the orch; it reaches `main` through commits made outside a landing, such as `4cb1c4b1` and `263b2ca1` committed by the orch, and shows on a pull request's own commits and comments.

## Wanted

- Every settings document `internal/shuttleengine/claudeengine/settings.go` composes, the run document and the creel-only document for an opened session alike, sets Claude Code's `attribution` to `{"commit": "", "pr": ""}`, so no session lym launches credits itself.
- The landing's credit block stays the one credit.
- The package doc and `docs/overview.md` state it beside the other settings each document carries.
