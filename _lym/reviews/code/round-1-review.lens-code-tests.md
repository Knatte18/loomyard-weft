```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviour → pinning test**

1. `importIssue` sets `EmptyBody` for a blank body → `TestImportIssueNewEntry` row "blank body flagged with the title" (`internal/boardengine/intake_test.go`, `wantEmptyBody: true`). Existing rows pin `false`.
2. `importIssue` sets `LocalPaths` from the body and comments: first-mention order, dedup, trailing punctuation trimmed, `wt/.lym/...` prefix kept, `_lym/` excluded → `TestImportIssueNewEntry` row "local paths in body and comments flagged". Existing rows pin `nil`.
3. `issueContent`'s blank-body note, with the title quoted and the body omitted when it is only white space (the check changed from `!= ""` to `TrimSpace`) → the same blank-body row, through `wantBody` and its `" \n"` body.
4. `issueContent`'s local-paths note and where it sits (after the link line, before the body) → the same local-paths row, through `wantBody`.
5. Fold path (`foldIssue` → `issueContent(issue, "### ")`, plus the flags set in `importIssue` after the fold branch) → **none**. `TestImportIssueFold` only uses `testIssue` and `testIssueWithComments`, whose bodies are non-blank and name no paths. The new-note rows cover the shared `issueContent` text. Nothing pins `EmptyBody` and `LocalPaths` on a fold result, or the note inside a `## From issue #n` section. The summary's claim that "a fold, whose section carries no title, keeps it too" has no test.
6. Already-recorded no-op leaves `EmptyBody`/`LocalPaths` unset → `TestImportIssueRecordedIsNoOp` asserts only `Recorded`. The CLI prints only `recorded` on that path, so nothing observable depends on the flags there. No finding.
7. `intakeImport` prints `empty_body` and `local_paths` only when the gap is present → `TestIntakeImport_FlagsEvidenceGaps`. Each row pins one key present and the other absent. Existing `TestIntakeImport_*` tests use the default body `"Body N."`, so they would print the keys if absence broke, but they don't assert absence. The new test's cross rows pin it.
8. `appendAttachments`, body then sections in order, body's trailing newlines trimmed → `TestRunCreate_Success` row "attachments follow the body" (`-b "details\n"`).
9. `appendAttachments` with a nil body (attachment alone is the body) → row "attachment alone is the body".
10. Section named by base name only → row "attachments follow the body" (`--attach ./report.md` renders `` `report.md` ``).
11. `attachmentSection` widens the fence past the longest backtick run → row "attachments follow the body" (`fenced.md` gets a four-backtick fence).
12. Unreadable attachment refused before transport → `TestCommand_RefusedBeforeTransport` row "missing attachment".
13. Over-limit body refused before transport, message naming the limit → row "attachment over the body limit".
14. Limit counted in runes and refused only when strictly over → **none**. The oversized fixture is `maxIssueBodyRunes+1` ASCII `x`, and the heading and fence push it further over. So `>=` in place of `>`, or `len(text)` (bytes) in place of `utf8.RuneCountInString`, still passes every row.
15. `-b ""` with `--attach` (an empty body is dropped, not joined as a leading blank section) → none. It's a minor edge, and its composition follows directly from the `*body != ""` guard, so no finding.
16. Help text, `doc.go`, `docs/overview.md`, `SKILL.md`, stencil → static prose, exempt.

**Findings**

- **LOW** · design · `internal/boardengine/intake_test.go` `TestImportIssueFold` (no fold row for a blank body or named `.lym/` paths). The fold path's import notes and its `EmptyBody`/`LocalPaths` result have no test. A regression that set the flags only in `importNote`, or that dropped the notes from a fold's `### `-level section, would pass. Evidence: `TestImportIssueFold` uses only `testIssue`/`testIssueWithComments` (`intake_test.go:166-211`). The flags are set after both branches at `internal/boardengine/intake.go:148-149`. The summary claims fold behaviour (`_lym/landing/summary.md`, "Importing" bullet one). Fix: add a fold call with `testIssueWithBody(n, "", …)` or a paths body to `TestImportIssueFold`, asserting the body and both flags.
- **LOW** · design · `internal/selfreportcli/cli_test.go` row "attachment over the body limit". The limit's off-by-one and its rune (not byte) counting aren't pinned: the fixture is far over the limit and pure ASCII. Evidence: `cli_test.go:143` (`strings.Repeat("x", maxIssueBodyRunes+1)`) and `cli.go` `appendAttachments`'s `utf8.RuneCountInString(text) > maxIssueBodyRunes`. Fix: make the oversized row's fixture multi-byte content whose byte count passes the limit while its rune count stays under it (accepted), and add a row exactly at the limit, or one past it.

Tool calls: none. The inherited context held every test and definition this lens needed.
