# Card 12 — intake-carries-comments

**Create:**
- `plan:internal/boardcli#importPointer` -> `func importPointer(slug string) string`
- `plan:internal/boardcli#isImportPointer` -> `func isImportPointer(body string) bool`

**Edit:**
- `internal/boardcli#intakeIssueView`
- `internal/boardcli#intakeList`
- `internal/boardcli#inboxIssue` -> `func inboxIssue(issue selfreportengine.Issue, comments []selfreportengine.IssueComment) boardengine.InboxIssue`
- `internal/boardcli#intakeImport`
- `internal/boardcli/intake_test.go`

**Uses:**
- `internal/selfreportengine#ListIssueComments`
- `internal/selfreportengine#IssueComment`
- `internal/boardengine#IssueComment`

**Intent:** Wire the comments through `lyx board intake` (D11).
`intakeIssueView` gains `Comments int`, tagged `comments`, and `intakeList` fills it from the issue's count, so `intake list` shows comments per issue with no per-issue call.
`intakeImport` calls `ListIssueComments` after `GetIssue` and before `ImportIssue`, so every network call stays outside the board lock and a failed fetch returns the error with nothing written; it converts the comments through `inboxIssue`, which now takes them and drops each one for which `isImportPointer` is true.
`importPointer` builds the comment lyx posts after an import, today an inline `Sprintf` in `intakeImport`, from one format constant declared beside it; `isImportPointer` is true only when the whole body, trimmed, equals that form with any slug between the backticks (a regular expression anchored at both ends built from the same constant), so a comment that quotes the sentence inside other text is kept verbatim.
`intake_test.go`'s fake inbox serves the comments endpoint with pagination; `TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages` asserts the `comments` count and that the fake saw no per-issue request, `TestIntakeImport_NewNoteCommentsAndCloses` and `TestIntakeImport_FoldIntoEntry` assert the body carries the comments oldest first across two pages at the right heading level, with a whole-body pointer comment absent and a comment quoting the pointer sentence kept, and `TestIntakeImport_RefusedBeforeAnyWrite` gains a row where the comment fetch fails and `board.json` is unchanged.

**ImpactSummary:** One helper re-signed with its single caller, two intake handlers extended; the import's board write and GitHub close are unchanged in order.
