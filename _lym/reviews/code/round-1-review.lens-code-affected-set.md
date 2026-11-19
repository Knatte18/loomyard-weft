```yaml
lens: code-affected-set
outcome: reported
```

My lens is code-affected-set: for each changed output, key, message or behaviour, find what reads or describes it, and flag any member the diff leaves wrong.

**Per change: the set found, covered or not**

1. **`intake list --full` `comments` key changes from an integer count to a list of `{body, created_at}`.**
   - `internal/boardcli/doc.go:182-184`: updated by the diff. Covered.
   - `list` long help and `--full` flag text, `internal/boardcli/intake.go:57-59,74`: updated. Covered.
   - `plugins/ly/skills/board/SKILL.md:233`: updated. Covered.
   - `TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages`, `intake_test.go:382-389`: asserts the new shape. Covered.
   - `contracts/orchrole/role-intake.md` fork outcomes: they now say the full list carries comments. Covered.
   - Readers of the old integer: none in Go code. A repo-wide grep for `intake list`, `comment count` and `comments (the` found no other reader, fixture or golden file. No help-text snapshot holds the flag string. Covered.
   - `docs/overview.md:472-476` describes intake without the `--full` shape, so nothing is stale. Covered.
2. **The `--full` path now makes a per-issue comments request, and a failed fetch fails the list.**
   - Test helper `fakeInbox.perIssueGets` and the per-issue assertion `intake_test.go:326-336`: updated in the diff. Covered.
   - The `CommentListFailure` row: added. Covered.
   - `internal/hubissues/hubissues.go:46`: the `Issue.Comments` doc ("comment count as GitHub reports it") is still accurate, and the code reads it as the fetch gate. Covered.
   - Role line 9, "never run it with `--full`": unchanged and still consistent, because the orch stays on the compact list. Covered.
3. **`carriedComments` is extracted and shared by `inboxIssue` and `intakeList`.**
   - Callers are `inboxIssue` (`intake.go:285`) and `intakeList` (`intake.go:213`).
   - Import's body text in the `import` long help (`intake.go:91-92`) still says "without lym's own import pointers". Covered.
   - Import tests at `intake_test.go:406-513` still pin the pointer drop. Covered.
4. **`role-intake.md` gains the comment-to-note split, widens the upsert limit and extends the one-line-per-issue report.**
   - The file is embedded through `contracts/orchrole/orchrole.go:20-34`, and no test asserts its prose: `internal/orchengine/prompt_test.go` has no intake match.
   - The `"kind":"note"` upsert it names matches `SKILL.md:203,214`, the existing way to record a finding as a note.
   - The report-line consumer is the orch, reading the fork's report under the same file's lines 20-21. It acts only on `failed` lines, so the extra trailing slugs break nothing. Covered.
   - Stencil wording: the widened upsert limit is licensed by the entry's "so the orch can split a comment … into its own note". No other rule in the file is dropped or weakened. Covered.
5. **`SKILL.md` import bullet now says the body carries comments under headings, and that `list --full` and the body leave out pointers.**
   - Matches `issueContent` (`internal/boardengine/intake.go:203-219`), which writes one heading per comment, and `carriedComments`. Covered.
6. **Repository rules, declared PATTERN changes, commit messages.**
   - No PATTERN change is declared, and the diff touches neither `PATTERN.md` nor `pattern/`.
   - The commit messages carry no `Pattern-Departure:` trailer.
   - The claims in commit b34e7406 match the code: comments are fetched only when the count is above zero, a failed fetch fails the list, and the result is oldest first with pointers left out.
   - Moving the changed per-issue assertion is a test question, which belongs to the code-tests lens and not this one.

**Findings:** none. Every member of the affected set I found is updated or still accurate.

**Tool calls:**
1. Grep for `intake list`, `comment count` and `comments (the` repo-wide, to find readers of the changed output beyond the manifest.
2. Grep for the role-intake rule text and its tests, plus `SKILL.md` lines 195-250. The manifest held only the diff hunk of `SKILL.md` and none of the orchrole tests.
3. Grep `internal/orchengine` and `contracts/orchrole` for intake references, to find code or tests that read `role-intake.md`.
4. Grep for the `--full` flag string, to look for help-text golden fixtures.
5. List `docs/reference` and grep `docs` for intake, to find user docs that might describe the old count.
6. Grep `docs/overview.md` and `docs/getting-started.md` for intake, to confirm what those docs say.
