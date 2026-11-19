# Intake import drops the issue comments

`intake-drops-issue-comments` · Task · bug, board · starting · Open-IDE

## What was seen

2026-11-19: `lym board intake import` of issue #153 recorded only its body; its two comments, each a further wish, were left on GitHub,
and `intake list --full` shows only a comment count. The orch had to fetch them with `gh` by hand after the operator asked.

## Idea

Import carries the issue's comments, other than intake's own pointer comments, into the entry's body under a heading of their own,
and `intake list --full` prints them, so the orch can split a comment that is a separate finding into its own note.
