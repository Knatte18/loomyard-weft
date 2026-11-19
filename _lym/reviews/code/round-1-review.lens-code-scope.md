```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to a goal point of the board entry, and each goal point to the changes that build it.

**Decisions.** A decision record alone is the answer key. It has no `Added after Discussion` entries.
- **G1:** import carries the issue's comments into the entry's body under a heading of their own, leaving out intake's own pointer comments.
- **G2:** `intake list --full` prints the comments.
- **G3:** the stated purpose. The orch can split a comment that is a separate finding into its own note.

**Per change**
- **C1:** `intakeList` fetches and prints the comments, using the new `intakeIssueView.Comments []intakeCommentView` and `intakeCommentView` (`internal/boardcli/intake.go:170-231`). This traces to G2.
- **C2:** the `carriedComments` helper is extracted and `inboxIssue` now calls it (`internal/boardcli/intake.go:269-287`). This traces to G2. C1 needs the same pointer filter import uses so that list matches import, so the change is forced by C1.
- **C3:** `doc.go`, the `list` long help and the `--full` flag text (`internal/boardcli/doc.go:183-184`, `internal/boardcli/intake.go:57-59,74`). Forced by C1, because the old text described an integer count that is now stale.
- **C4:** the `plugins/ly/skills/board/SKILL.md` intake bullets (diff 003, lines 9-15).
  - The `--full` line is forced by C1.
  - The import sentences document G1's behavior, which was already present at the base but undocumented in the skill. This traces to G1, as documentation of the existing behavior.
- **C5:** `contracts/orchrole/role-intake.md`, the comment and split-note rule plus the widened upsert limit (diff 001, lines 9-18). This traces to G3.
  - The report-line addition "then the slug of each note split from a comment" (diff 001, lines 22-23) is forced by C5's new outcome.
- **C6:** `TestIntakeList_…`, which uses `threadWithPointer`, adds the `CommentListFailure` row and changes the per-issue-request and comments assertions (`internal/boardcli/intake_test.go:258-388`). Forced by C1 and C2.
- **C7:** commit 57df7420 re-breaks the wrapped comments in `intake.go`. It touches only lines C1 to C3 add, so it is forced by those changes.

**Per decision**
- **G1:** the code was already at the review base. `issueContent` in `internal/boardengine/intake.go:205-220` writes one `Issue comment, &lt;date&gt;` heading per comment. `boardengine` is not in the diff, and commit 64e58795 is an ancestor of base c5a0ea74. The base's `inboxIssue` already dropped pointers (diff 002, lines 109-116). In the diff, G1 is built by C2, a filter refactor that keeps the behavior, and by C4's documentation. `_lym/landing/summary.md`, under Design decisions, says this openly.
- **G2:** built by C1, C2 and C3, with C6 as its tests.
- **G3:** built by C5.

**Findings:** none. Every change traces to G1, G2 or G3, or is forced by a change that does. Every goal point is either built by a change or was present at the base, and the change description says so.

Tool calls: none. The diff pieces, the decision record, `_lym/landing/summary.md`, `internal/boardengine/intake.go` and the ancestry check of 64e58795 were already in my context.
