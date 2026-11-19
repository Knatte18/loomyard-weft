---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: contracts/orchrole/role-intake.md:36
    summary: The comment-to-note split is not settled once per issue by the CLI, so two overlapping intake forks can each upsert a note for the same comment, contradicting the role's claim that a second fork is harmless
    fix: In contracts/orchrole/role-intake.md, in the bullet that opens "The full list carries each issue's comments", make the split conditional on the fork's own import of that issue having written the board, which is a success or an error carrying a close payload, and add that a split is skipped when the import prints recorded. Leave the close re-run rule and the second-fork paragraph unchanged.
    origin: assist:assist
  - id: F2
    severity: LOW
    class: design
    location: internal/boardcli/intake.go:209
    summary: A failed comment fetch in list --full returns the hubissues error, which names the repository but not the issue whose comments failed
    fix: In intakeList, at the ListIssueComments error return, wrap the message with the issue number, for example outputError(out, fmt.Sprintf("issue #%d: %v", issue.Number, err)), and extend the CommentListFailure row's error assertion to check for the issue number.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/boardcli/intake_test.go:290
    summary: The CommentListFailure row asserts only that some error message exists, so a failure from another step passes it; the import test for the same failure pins the comment-list error text
    fix: In TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages, add a wantErr string field to the table (empty for rows that do not check it), set it to "issue comment list failed" on the CommentListFailure row, and in the wantCode != 0 branch assert strings.Contains(msg, tt.wantErr) when wantErr is set.
    origin: lens:code-tests
---

The diff makes the board entry's `list --full` half and the role's split rule.
Import's carrying of comments under headings was already at the review base, since 64e58795 is an ancestor of the base, and the change description says so.
The shared `carriedComments` filter leaves import's behavior unchanged.
`list --full` prints the same comments import carries, oldest first, pointers dropped, and `[]` rather than null.
The compact path still makes no per-issue request.
The changed per-issue assertion is not loosened: it still forbids every per-issue GET outside the full view's `/1/comments`.

### [MEDIUM] Overlapping intake forks can duplicate split notes

**Location:** contracts/orchrole/role-intake.md:36

**Issue:** The role says a second fork beside the first is harmless because the CLI settles each issue once: a second import prints `recorded`.
The new rule sends every fork that reads an issue's separate-finding comment to `lym board upsert` a note, and nothing ties that upsert to the fork's own import.
Two forks that pick different slugs create two notes for one comment, and no issue-level check stops the second.

### [LOW] Comment-fetch error in list --full does not name the issue

**Location:** internal/boardcli/intake.go:209

**Issue:** `ListIssueComments` errors read "github issue comment list failed on <repo>: …" with no issue number, and `intakeList` returns that message unchanged.
With several issues in the list, the orch, which handles this error as a lym bug, cannot tell which issue's fetch failed.

### [LOW] CommentListFailure row does not pin the failure it names

**Location:** internal/boardcli/intake_test.go:290

**Issue:** The row checks exit 1, `ok:false`, a non-empty error and no `issues` key.
Any error earlier in the handler gives the same result, so the row would pass even if the comment fetch were not what failed.
The import test's comment-fetch row at intake_test.go:567 asserts "issue comment list failed".

## Rejected

- lens:code-tests finding 1, no row for an issue whose comments are all import pointers: the `[]` comes from the initialization at intake.go:207, which the issue-5 assertion already pins; the finding rests on a hypothetical refactor, not a current gap.
