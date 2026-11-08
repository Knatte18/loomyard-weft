```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches for F1 and F2 and found no findings. F3 is a coverage gap, not a set under one key, so I left it unchecked.

- **F1:** the claim "the row waits on the orch's answer" or "does not wait" appears in `internal/landingshed/doc.go:85-88`, `internal/landingshed/prreview.go:334-336` and `docs/overview.md:1249`. All three now say the row asks and does not gate. `contracts/orchrole/orch-role.md:192-198`, `contracts/stencils/landing/landing-template-pr-review.md:52-56` and `internal/loomengine/template.yaml:46` make no claim about gating, so they agree. Nothing else in the repo states the opposite.
- **F2:** the old claim "a failed write of the ask is an error" is gone from `prreview.go` and `doc.go`, where it was the only place it appeared. `ParentReviewAsked` is set only in `askParentReview` at `prreview.go:360`, so no other write site needed the same change. The only `askParentReview` caller is `openPullRequest` (`prreview.go:323`), which still checks an error that is now always nil.

**Calls made**
1. `grep` for wait/gate claims about the orch's answer across `.go`, `.md` and `.yaml` files: my context lacked a repo-wide search for sites that state the claim.
2. `grep` for `parent_review_asked`, `ParentReviewAsked` and `pr_review_ask_parent` outside tests: my context lacked a repo-wide list of the flag's write and doc sites.
