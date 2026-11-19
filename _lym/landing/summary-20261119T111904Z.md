# intake list --full prints each issue's comments

`lym board intake list --full` printed a comment count and nothing more for each issue,
so the intake fork could not see a comment that carries a further finding without fetching it from GitHub by hand.

## What changes

- `intake list --full` now prints `comments` as a list of `{"body":…,"created_at":…}`, oldest first.
  lym's own import-pointer comments are left out, the same filter import uses.
  This replaces the old integer count under the same key.
- Comments are fetched only for an issue whose GitHub comment count is above zero,
  so the compact `list` and an issue without comments make no extra request.
  A failed comment fetch fails the whole list with the GitHub error.
- `boardcli`'s import-pointer filter is now one helper, `carriedComments`, which import and list share.
- The orch's intake role (`contracts/orchrole/role-intake.md`) lets the fork turn a comment that is a separate finding into a note of its own
  through `lym board upsert` with `"kind":"note"`, and the fork's per-issue report line names each such note.
- The `boardcli` package doc, the verb's help and the `ly:board` skill describe the new `--full` output.

## Design decisions

- The entry's other half, import carrying the comments into the entry's body under headings of their own,
  had already landed in 64e58795, so this change covers only `list --full` and the fork's instructions.
- `comments` changes type from a number to a list instead of adding a second key.
  The count can be derived from the list, and the fork is the only reader of `--full`.

## Tests

`TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages` now checks the full view's comment list:
two pages of comments, with a whole-body pointer dropped and a quoted pointer kept.
It also checks that an issue without comments prints `[]`, and that only the full view requests comments.
A new `CommentListFailure` row checks that a failed comment fetch gives an error envelope.
