```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none.

The task's test changes only retarget citation strings in comments and failure messages. They add no test, row or assertion, so test economy raises no finding.

| Behaviour | Pinning test | Result |
|---|---|---|
| `recipe-owns-failures` section in `internal/landingshed/doc.go` (doc comment, line 7) | None | Exempt. It is static prose, and the decision record's enforcement for this rule is only the doc. |
| Merged `landingshed/doc.go` line 123 ("first failing tier") from the parent | The parent's own tests in `internal/gatecli` and `internal/landingshed`, not this task's | Outside this diff. |
| `docs/overview.md` regrouped references and the merged gate line | `internal/lyxcwd/docslink_test.go` | Pinned. It resolves every `docs/` link, and the diff touches that file in only 3 lines. |
| `PATTERN.md` has 30 entries, one background file each, no orphan | `internal/pattern`'s `pattern.Check` test, which the diff leaves untouched | Pinned by the existing check. The `pattern/` file count is 28 and `PATTERN.md` has 30 entry lines. That mismatch is for the pattern-fit lens to settle against `pattern.Check`. |
| Retargeted enforcement-test citations (`cmd/lyx/*_test.go`, `internal/*/..._test.go`, `contracts/stencils/*_test.go`, `tools/sandbox/pathresolve_guard_test.go`) | Each test is its own guard, and its assertions are unchanged | Held. The diff changes only message text, with no new row or duplicate coverage. |
| Failure messages in `cmd/lyx/configstrictness_test.go`, `destructiveguard_test.go`, `verifiedtree_test.go` | The same tests | Held. The messages now name a surviving ID, and the comment rewraps there are semantic-line-break edits. |

I found nothing that raises a test-economy finding:
- No new test duplicates an existing one.
- No per-helper test was added.
- No review-fix test was added for a non-coverage finding.
- No assertion was added on stencil prose.
- No behaviour is left uncovered. Every behaviour the diff adds is either a doc or a citation retarget.

Tool calls (1 of 6):
1. `git diff 5e36064..HEAD --stat` filtered to test files, plus the added and removed lines of `*_test.go`. My context held no listing of the test files changed in the range.
