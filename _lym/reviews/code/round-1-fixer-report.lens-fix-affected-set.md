```yaml
lens: fix-affected-set
outcome: reported
```

No findings. The F8 comment set is complete, and the other sets the fix touches are covered.

- **F8:** the grep for the stale "git push and gh" wording leaves no stale member. The three named comments are reworded: `internal/shuttleengine/spec.go:49`, `internal/shuttleengine/claudeengine/settings.go:322` and `internal/shellguard/rows.go:23`. Every other site that describes the publish deny already uses the new wording: `internal/shellguard/doc.go:30`, `internal/shuttleengine/claudeengine/doc.go:266` and `internal/landingshed/doc.go:33`.
- **F7:** nothing outside `rows.go` and its test references the removed gh commands (`gh issue`, `gh repo view`, `gh run list`, `gh run watch`, `gh search`, `gh status`, `gh pr status`). The one hit, `internal/selfreportengine/selfreport.go:114`, is a historical comment unrelated to the guard. The stencil line at `contracts/stencils/landing/landing-template-pr-review.md:40` names only `gh pr view`, `list`, `diff`, `checks`, `gh run view` and `gh api` reads, which the cut allowlist keeps. Both consumers of the switch (`internal/shuttleengine/claudeengine` and `internal/shuttleengine/codexengine`) read the same generated steer, so neither needs a separate edit.
- **F9:** `pollFailure` has two callers, `internal/landingshed/prreview.go:347` and `:693`, and both pass the new `*bool` argument. No test references `pollFailure` or the old warning text. The only test hit on the old text is the row the fixer already updated.
- **F1, F3, F4:** these sites are all in the generated pattern, and every `gh` alternative in `publishCommandPattern` was updated. This fork did not run the pattern.

Calls made:
1. Grep for the stale "push and gh" wording across the repo, to rerun F8's search.
2. Grep for the removed gh commands in non-test files, to rerun F7's search (combined with the first call).
3. Grep for `pollFailure`, `transientWarned` and the old warning text, for F9's callers (same call as 1 and 2).
4. Grep for `DenyPublish` and publish-deny wording, to find other sites that describe the deny (my context only held the files the review named).
5. Read the comment lines around the other `DenyPublish` sites in `samplerengine`, `claudeengine/doc.go` and `landingshed/prsession.go`, to confirm none is stale.
