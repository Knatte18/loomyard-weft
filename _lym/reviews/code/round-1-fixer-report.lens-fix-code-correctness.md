```yaml
lens: fix-code-correctness
outcome: reported
```

Findings (fix-code-correctness):

1. **LOW**, `internal/landingshed/prreview.go`, `askParentReview` signature.
   - The function now returns `nil` on every path, so the `error` return is dead.
   - The caller `if err := p.askParentReview(rec); err != nil { return ... }` at `openPullRequest` is now unreachable code.
   - Behaviour is correct. After a post that succeeds and a record write that fails, the flag stays unset, the warning is logged, the row goes on to `wait`, and a later entry asks again. This matches the doc comment.
   - Dropping the return value and the caller's error branch would follow the code-quality rule "no code path for something the code's own guarantees already prevent".

2. **LOW**, `internal/loomcli/landingdeps_test.go`, row `the hub layer set to true reads true`.
   - `seedLoomConfigWithKeys(t, loc.AnchorPath(), ...)` writes the worktree's own `loom.yaml`, not the hub board-dir layer. The row name is misleading.
   - The assertions are sound anyway:
     - A closure that read only the board-dir layer would get `false` on this row, since `hubLocation` seeds the template default there. The row would fail.
     - A closure that ignored the local layer would get `false` on the third row, which sets the anchor layer to `false` and the local layer to `true`. That row would fail too.
   - Both layers are therefore pinned. Only the label is wrong.

3. **LOW**, F2 has no regression row.
   - The review's fix asked for a row asserting a single ask after a failed record write, or else keeping the error.
   - The fixer did neither, and the fixer-report should state that. The fixture has no seam to fail the record write alone.
   - If a row were added by removing the record inside the ask seam, it would pass with the old defect in place too, because the wait's record read would also error.

Checked with no problem found:
- The template line `pr_review_ask_parent: false` exists at the start of a line, so the `seedLoomConfigWithKeys` replacement matches and its not-replaced check does not fire.
- `prePushCheck{}` is never run, because `guardedPush` only wraps the push, so passing the zero value is safe.
- No existing test expects an error from a failed ask-record write. The `prreview_session_test.go` rows I read cover only a failed post and an unrecorded orch.

Tool calls made:
1. `grep` of `internal/loomengine/template.yaml` and `sed` of `internal/loomcli/wiring_test.go:875-890`. My context lacked the template key line and the tail of `seedLoomConfigWithKeys`, which decide whether the new test's seeding is valid.
