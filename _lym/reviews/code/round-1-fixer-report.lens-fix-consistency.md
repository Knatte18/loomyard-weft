```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: two stale twins found, both in `_lym/landing/summary.md`.

- **LOW, `_lym/landing/summary.md`, `## What changes` → "Readers of the state" (line 26).**
  - The summary says "The autonomous PR-Review block tells the session that an undelivered end is a refusal, not a down parent".
  - F3 changed the stencil to say "refused or failed to deliver the ask … its `reason` names why".
  - The summary's own `## What changes` bullet on `burlerengine` (line 20) already says "refused or that failed", so the summary now disagrees with itself and with the stencil.
  - The fixer edited only the `## Merge danger` section of this file.
  - Fix: reword line 26 to "refused or failed to deliver", as the stencil does.
- **LOW, `_lym/landing/summary.md`, `## Tests` (line 44).**
  - The summary says `TestPRReviewAsker_AskReachesTheParentSignedAsTheCaller` "drives `prReviewAsker`'s ask".
  - F1 changed the test to drive `parentReviewPRAskFunc(pair, tt.claim)(rec, "head-1")`, the verb's seam.
  - The bullet should name `parentReviewPRAskFunc`.
  - Line 46 also says "With no claim, it is signed as `LOOM`", which still holds.

Other checks came out consistent:
- **Test comment:** the test's doc comment (`prreviewask_integration_test.go:22`) matches the new seam.
- **Stencil and verb wording:** the stencil and the `ask` verb's Long text now both say "refuses or fails to deliver", as `internal/landingshed/prreviewask.go:44` does.
- **Remaining refusal wording:** the grep (excluding tests and the review files) found no other copy of the "creel refused" wording.

Tool calls: one `grep -rniE` over `*.md` and `*.go` for "creel refus|refusal as|undelivered". My context lacked any search for twins of the changed wording.
