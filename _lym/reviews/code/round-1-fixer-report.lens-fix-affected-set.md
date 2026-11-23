```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F3:** Only three places word an `undelivered` end around creel refusing the ask. They are the stencil line, the `ask` verb's Long text and the `ParentAskUndelivered` doc. All three now say "refused or failed". I found no other prose that narrows `undelivered` to a refusal.
- **F1:** `prReviewAsker` has two call sites outside the test. The PR-Review row's first ask still passes `""` at `landingdeps.go:478`. The verb path now goes through `parentReviewPRAskFunc(target, strand)` at `parentreview.go:407`. The integration test drives `parentReviewPRAskFunc` directly, so no caller was missed.
- **F2:** Its premise is a record format, not a set of sites, so I did not re-run a search for it.

Limit: the test calls `parentReviewPRAskFunc` directly and not `preRun`. A regression that makes `preRun` pass `""` to `parentReviewPRAskFunc` would still pass. That is the shape the review's fix proposed, so I did not raise it as a finding.

Tool calls used:
1. `grep -rniE "undelivered"` filtered to "refus" over non-test files: F3's set of places wording `undelivered` as a refusal.
2. `grep -rn "prReviewAsker(\|parentReviewPRAskFunc("`: F1's set of callers, which my context lacked after the edits.
