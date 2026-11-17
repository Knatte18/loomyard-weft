```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens on F1 (commits after the merge, comments rewritten by them).

I reran F1's search with `git log bae0ec63..HEAD` and `git show 382a591f`. The fix covers every member of the set.

- **Commits after the merge:** exactly one, `382a591f`. The summary's "followed by one commit, `382a591f`" matches.
- **Comments rewritten:** exactly three, as the summary lists them:
  - the `overview.go` file header, which was a 124-character line;
  - the `overviewPlace` doc, which broke after a comma;
  - the `TestRenderOverview` doc in `overview_test.go`, which broke after commas in two places.
- **Files touched:** only `internal/heddleengine/overview.go` and `internal/heddleengine/overview_test.go`, with comment lines only, so "leaves the code untouched" holds.

One LOW finding. The changed text is the third bullet of `_lym/landing/summary.md`, "broke two blocks after a comma mid-sentence". The diff shows two comment lines in that doc, each ending in a comma (after "marked," and after "left out,"). The phrase is slightly garbled, but it names the right set and no reader would act on it differently.

Tool calls: one, a single Bash call running `git log` and `git show` on the post-merge range. My context had the review but not the commit's diff.
