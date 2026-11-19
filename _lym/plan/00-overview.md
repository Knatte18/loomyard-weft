---
format: 5
approved: true
language: go
---

# Plan: board viewer, its server and the VS Code keys

The operator gets an interactive board page opened from VS Code by one key.
The board renders a view projection (`board-data.js`) and an embedded page (`board.html`) beside the README, committed and replayed with it.
A per-hub loopback server, `lym board serve`, serves the page, the projection, its live changes and the page's writes, each write checked against the entry revision the page read.
`lym board open` makes sure the server runs and opens the page; the `board open` worktree task, the `beam start` workspace task and two keys reach both from VS Code.

## Card Index

1 — board-view-projection — render board-data.js from the README's own model, with entry and board revisions and Board.View
2 — entry-revision-check — refuse a write whose expected entry revision is stale, inside the write section
3 — viewer-page-asset — embed board.html, render it, and claim both files in the Bolt replay
4 — board-server-routes — boardserve's HTTP handler: page, board, long poll, file route, writes and their checks
5 — board-server-lifecycle — single-instance lock, state, launcher, watch loop, idle and binary exits
6 — board-serve-open-verbs — lym board serve and lym board open, the opener and the runproc role
7 — vscode-tasks-and-keys — board open worktree task and the alt+shift+9 and alt+shift+0 keys
8 — workspace-content-seam — fabric's opaque workspace-content hook and vscode's member splice
9 — beam-start-workspace-task — the ide hook writing beam start into the hub and beam workspace files

## Shared Decisions

- `Render` keeps its signature; `renderFiles(stored, rendered, ...)` lets `RenderToDisk` take each entry's revision over the stored entry while file names follow the copy that clears a missing record's `Discussed` (D1), with no caller of `Render` re-signed.
- The revision check is a `Board.Expecting(slug, revision)` copy of the board whose write section checks it after the load migrations; the CLI's board never carries one, so its verbs behave as today (D8).
- The server checks a release's, draft's and undraft's kind and status rule against the entry of a `Board.View()` whose revision it then has the write section re-check, so the rule and the write see one entry state.
  Every op but a create refuses an empty revision before any write.
  An edit carries its revision straight to the write section;
  a promote first refuses a task by its kind in the current `Board.View()`, then carries its revision to the write section.
  A create checks only that its slug is absent before the write section checks it again.
- Route paths are named constants in `internal/boardserve`; the static page spells them, and the server's test pins that the page holds each.
- The server's lock file sits in its state directory, as D11 states and as creel's daemon lock does, and is taken only through `internal/lock`'s functions.
- The opener is waited on by one goroutine whose `cmd.Wait` feeds the result channel, as creel's daemon spawn does, instead of `proc.Reap`, because a failed opener's error (D5) needs the exit status `proc.Reap` discards; `PATTERN-child-processes` admits either wait.
- The record has no `## PATTERN changes`; card 5 extends `PATTERN-daemon-wakeups`' background list only.
