# intake list --full prints each issue's comments

`lym board intake list --full` now prints each issue's comments, so the intake role can read the further findings an issue's thread holds.
Before, it printed only a comment count, and the orch had to fetch the comments with `gh` by hand.
Issue #153 showed this: its two comments were each a further wish, and the import recorded only its body.

## Changes

- `internal/boardcli/intake.go`: the `comments` field of the full view changes from a count to a list of `{"body":…,"created_at":…}`, oldest first.
  The list leaves out lym's own import pointer comments, the same ones the import already drops from the entry's body.
  `carriedComments` is now the one filter for both the list and `inboxIssue`.
- The list fetches an issue's comments only in the full view, and only for an issue GitHub counts comments on.
  The compact view makes no per-issue request.
  A failed comment fetch fails the command with an error that names the issue.
- `contracts/orchrole/role-intake.md`: the intake fork may split a comment that is a finding apart from its issue into a note of its own.
  It writes the note with `lym board upsert` and `"kind":"note"`, quoting the comment and naming the issue, and returns the slug on the issue's line.
  It splits only when its own import of that issue wrote the board, so a re-run that prints `recorded` splits nothing.
- `plugins/ly/skills/board/SKILL.md` and `internal/boardcli/doc.go` describe the new `comments` shape.

## Design decisions

- `comments` changes type in place instead of a new field beside the count.
  Its consumers are the intake role and the `ly:board` skill, both updated here; any other reader of the count needs to move to `len(comments)`.
- The split guard keys on the fork's own import writing the board, since a second fork reading the same list cannot otherwise tell that a note for the comment already exists.

## To check by hand

- Several forks that each split the same issue's comments can write duplicate notes: the guard in `contracts/orchrole/role-intake.md` covers a re-run of one fork, not two forks that race.
  Check whether the role needs a dedupe by slug or by quoted comment.
- Nothing was run against a live GitHub repository: try `lym board intake list --full` on a repo with an issue that has comments, including an already imported one.
