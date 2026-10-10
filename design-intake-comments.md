# Board intake drops issue comments

`intake-comments` · Note · bug, board, selfreport

`lyx board intake import` writes only the issue title and body (`issueContent` in `internal/boardengine/intake.go`); `selfreportengine` never fetches comments, so every comment on an issue is lost at import.
The operator treats a comment as part of its issue: #479 and #480 carried as much in comments as in their bodies and were completed by hand on 2026-10-10.

## Expected

- `import`, for a new note and for `into`, appends each comment after the body under a `## Issue comment, <date>` heading, oldest first.
- `intake list` shows the comment count, so an orch sees that an issue has comments before it decides how to import it.
- The pointer comment lyx itself posts on import is never carried back in.
