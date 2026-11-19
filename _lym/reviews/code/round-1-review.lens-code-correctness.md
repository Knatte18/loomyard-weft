```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes with one concrete input.

**Function and branch list**
- **`intakeList`, compact (`full=false`), fullInbox:** it appends the summaries and never reaches `ListIssueComments`. Correct: it makes no per-issue request (intake.go:203-205).
- **`intakeList` full, recorded issue #4:** `continue` comes before any fetch, so it makes no request. Correct (intake.go:200-201).
- **`intakeList` full, `issue.Comments == 0` (#5):** `comments` stays `[]intakeCommentView{}` and marshals to `[]`, not `null`. Correct (intake.go:207-208).
- **`intakeList` full, `Comments=3` over two pages, middle comment a whole-body pointer:** `ListIssueComments` follows pagination, and `carriedComments` drops the pointer and keeps the quoted one, oldest first. Correct (intake.go:209-215, hubissues.go:121-145).
- **`intakeList` full, `Comments&gt;0` where every comment is a pointer:** `carriedComments` returns nil and the range over nil adds nothing, so it prints `[]`. Correct.
- **`intakeList` full, `ListIssueComments` fails:** it returns `outputError` and prints no issues. The error is passed on, not swallowed, and no state is written. Correct, but the message lacks the issue number (finding F1).
- **`intakeList` full, stale count (`Comments=0` while comments exist):** it prints `[]` without fetching. That follows the stated contract ("only for an issue GitHub counts comments on"), and GitHub's count is authoritative. No finding.
- **`intakeCommentView`, `CreatedAt` time.Time:** it marshals as RFC3339. The import heading uses the UTC date of the same value, so both read the same instant. Correct.
- **`carriedComments(nil)` and `carriedComments` with an empty slice:** both return nil. Correct.
- **`carriedComments`, trimmed whitespace body " Imported into the board as `old-slug`.\n":** `isImportPointer` trims, so the comment is dropped. Correct, and the same as the base.
- **`inboxIssue`, comments all pointers or none:** `carried` stays nil and `InboxIssue.Comments` is nil, the same as the base's loop gave. `issueContent` ranges over nil and adds no sections. Behaviour is unchanged (intake.go:283-298, boardengine/intake.go:217-219).
- **`inboxIssue`, mixed thread:** the order is preserved and the body and created_at are copied. Correct and unchanged.
- **`intakeImport` with failing comments:** this path is unchanged and still fails before the board write.
- **Test `fetchesComments` predicate, every row:** I checked `tt.issues[0]` for all rows. `Full`: issue #1 thread, fetches expected. `EmptyFull`: #2 has no thread, none expected. `CommentListFailure`: one GET of `/1/comments` expected. The `Repo*` rows are not full and the repo is never opened, so none expected. No row panics, and every row evaluates correctly (intake_test.go:327-336).

**Findings**

F1
- **Severity:** LOW
- **Class:** design
- **Location:** internal/boardcli/intake.go:209-211
- **Summary:** When `ListIssueComments` fails inside `list --full`, the error names the repository but not which issue's comments failed. With many issues in the list, the operator cannot tell which fetch broke.
- **Evidence:** `classifyCallError` builds `"%s on %s: %s"` from apiFailure, `r.FullName()` and the message, and takes no number (internal/hubissues/hubissues.go:196-210). intake.go:211 returns `err.Error()` without wrapping it with `issue.Number`. The import path is not affected, because its caller names the issue in its payload.

Tool calls:
1. `grep` of `classifyCallError` in internal/hubissues/hubissues.go, to see whether the comment-list error carries the issue number. My context held only `ListIssueComments`, not the error builder.
