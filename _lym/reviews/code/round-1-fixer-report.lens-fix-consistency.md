```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the change set. Siblings and twins of the changed text.

1. **LOW** — `internal/landingshed/prreview.go:29`, the `PRReview` type doc: "sets the title and body again when the change description changes".
   - F10 reworded the same claim in the `wait` doc comment (line 456) to "the text yielded from the change description, the regression check and the credit block".
   - The type doc was left with the old trigger.
   - Evidence: `pullRequestText` builds the text from all three, and `syncText` compares that whole text.

2. **LOW** — change description (the "Testing" section and the design note on the broadened read set), outside the code write surface.
   - F7 cut `githubAllowed` to `api`, `pr` (checks, comment, diff, edit, list, review, view) and `run view`.
   - The description still records the broadened set (`gh issue`, `repo view`, `run list` and `watch`, `search`, `status`, `pr status`) as a design decision, and claims an allow row for every listed form.
   - The test rows now deny those forms. `gh_search` flipped from false to true, and the new rows `gh_run_watch`, `gh_status`, `gh_pr_status`, `gh_issue_list`, `gh_repo_view` and `gh_run_list` are denies.
   - The fixer should say in its report that the description needs this update, since F6 and F7 point opposite ways on those forms. F6's own fix key says to correct the Testing section.

Other siblings agree with the change set:
- `shellguard/doc.go:30-31`, the publish row's guardrail comment, the stencil's `gh` list and the generated steer all match the new table.
- The stencil's `gh run view` stays allowed.
- The three F8 comments now use `doc.go`'s wording.
- Both `pollFailure` callers pass a flag pointer.
- The two tests that counted the old warning text now use the new text.

Tool calls: none; my context already held every file I checked.
