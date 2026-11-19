```yaml
lens: code-tests
outcome: reported
```

Task: work the code-tests lens over the diff, listing each behaviour the diff adds or changes and the test row that pins it.

**Per behaviour, the test that pins it**

1. **`list --full` prints `comments` as a list of `{body, created_at}`, oldest first, across pages.** Pinned by the `Full` row of `TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages` (`internal/boardcli/intake_test.go:382-386`). `threadWithPointer` spans two fake pages, and the expected string fixes both the order and the dates.
2. **Import-pointer comments are left out, and a quoted pointer is kept.** Pinned by the same `Full` row (`intake_test.go:382-383`). The whole-body pointer is absent and `quotedPointerComment` is present.
3. **An issue whose GitHub comment count is 0 prints `[]`, not null.** Pinned by the `Full` row's check on issue 5 (`intake_test.go:387-388`); a nil slice would print `&lt;nil&gt;`.
4. **Comments are fetched only when `issue.Comments &gt; 0`.** Pinned by the `Full` row's per-issue loop (`intake_test.go:327-336`). Without the guard, `/5/comments` and `/6/comments` would be requested and the loop would flag them.
5. **The compact `list` makes no per-issue request.** Pinned by the `Compact`, `EmptyCompact` and `ListFailure` rows, where `fetchesComments` is false and any GET is flagged (`intake_test.go:327-333`).
6. **A failed comment fetch fails the whole list.** Pinned by the `CommentListFailure` row (`intake_test.go:290`, `337-345`): exit 1, `ok:false`, a non-empty error and no `issues` key.
7. **Import still filters pointers through the shared `carriedComments` (the `inboxIssue` refactor).** Pinned by the import tests that use `threadWithPointer`, which expect the body's comment sections (`intake_test.go:406`, `432`, `487`, `513`). The reach report lists `TestIntakeImport_NewNoteCommentsAndCloses`, `TestIntakeImport_FoldIntoEntry` and others for `carriedComments`.
8. **An issue with `Comments &gt; 0` whose comments are all import pointers prints `[]`.** None. No row serves such an issue in the full view.

**Loosened checks:** none. The old assertion "no per-issue request in any row" still holds in full for every non-full row. In full rows it is relaxed only to the `/1/comments` fetch the feature adds, and it still fails on any other per-issue GET. The old count assertion (`2` and `0`) is replaced by a stricter content assertion, not widened. The test still checks what its name states: pull requests, recorded and closed issues excluded across pages.

**Findings**

1. **LOW**, class `design`, at `internal/boardcli/intake_test.go:257-264`. No row covers an issue that GitHub counts comments on but whose comments are all import pointers. That is the one path where `carriedComments` returns nil inside the fetch branch, so the "`[]` rather than null" guarantee there is unpinned. Today `[]` comes from the initialisation at `internal/boardcli/intake.go:207`. A refactor that set `comments` straight from the converted result would print `null` and no test would fail. Evidence: `intake.go:207-216` and the `fullInbox` rows at `intake_test.go:257-264`, where only issue 1 has a thread and it holds non-pointer comments.
2. **LOW**, class `design`, at `internal/boardcli/intake_test.go:290`, `341-343`. The `CommentListFailure` row asserts only that some error message exists. Neither the GitHub comment-list error, which the commit message and `summary.md` say the list fails with, nor any other specific error text is pinned. A failure from an unrelated step would also pass. Evidence: `intake_test.go:341-343` and `hubissues.go` `ListIssueComments`, which classifies the error as "github issue comment list failed". The import test at `intake_test.go:567` asserts "issue comment list failed" for the same failure, so the precedent exists.

Tool calls: none.
