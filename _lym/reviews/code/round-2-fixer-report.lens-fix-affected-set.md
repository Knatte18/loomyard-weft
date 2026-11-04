```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each finding, one tool call per finding, to check the fix reached every site in the set.

**Findings**

1. **LOW** — `internal/pullrequest/doc.go:4`, outside the diff and not named by the review.
   - Summary: the package doc still says the record holds "whether the row ended the review on that flag".
   - Evidence: this is F2's gap. `record.go:35` is now fixed, but `doc.go:4` keeps the old wording, though Finalize's `openAtOnce` also writes `ReviewEnded: true`.
   - Fix: the package doc should say the review ended either by the row on that flag or by Finalize on the record it opens itself.

2. **LOW** — `internal/landingshed/finalize.go:172-173`, the comment above Finalize's landing steps.
   - Summary: the comment says a parent that takes its pull request at once "reopens a closed one", with no sign of the new timing.
   - Evidence: `doc.go:261` and `finalize_pr.go` now say that reopen runs in `mergeOnGitHub` after the owner checks. The operator's-ending reopen on the next line still runs in `pullRequestLanding`, before them.
   - Fix: the comment should not claim both reopens happen at the same step. It is a stale-sibling nit, not wrong.

**Searches that found no gap**
- F1, `.Reopen(` and `prLanding{` constructions:
  - The only production reopen sites are `finalize_pr.go:214`, the non-MergeAtOnce `pullRequestLanding` reopen, and `:272`, the new one in `mergeOnGitHub`. `prreview.go:828` is a different, request-side `pr.Reopen()`.
  - Every `prLanding{}` construction sets `reopen` correctly. Only the MergeAtOnce closed case sets it, and the `:206` open and `:220` merged cases don't need it.
- F2, wording about who writes the ending mark: the only other site that says only PR-Review writes it is `pullrequest/doc.go:4`, covered in finding 1. `PATTERN.md:38` names Finalize as allowed to reopen, so no conflict.

**Tool calls**
- `grep` for `.Reopen(` and `prLanding{` in non-test Go files under `internal`, because my context held only `finalize_pr.go`.
- `grep -i` for the ending-mark wording across Go and markdown files, to cover F2's sites beyond the two the review named.
