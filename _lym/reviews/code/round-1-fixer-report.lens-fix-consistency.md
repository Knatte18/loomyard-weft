no findings

- Role wording (F1): the new split bullet agrees with the second-fork paragraph, the close re-run bullet and the report-line bullet; the board skill states no split rule.
- Error twin (F2): `intakeImport` returns the comment-list error without the issue number, which is consistent because the import payload already names the issue; the import row asserts only "issue comment list failed".
- Table rows (F3): every row carries the trailing `wantErr`, empty on all but `CommentListFailure`; the `strings` import already existed.

Tool calls (3):
1. grep for sibling `ListIssueComments` call sites, the split-rule wording and `list --full` references: the context lacked them.
2. Read of `intake.go:315-335`: to see the import-path error handling.
3. Read of the board skill's intake section: the context did not show it.
