```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text still agrees with its siblings and twins.

**MEDIUM** — `internal/landingshed/doc.go:43`, package doc for the rule read
- **Summary:** the doc says "any other failure, an unusable token included, logs a Warn and reads as no rule". After the fix that holds only for a run with no recorded pull request.
- **Evidence:** `resolveWithParentRule` (`pullrequestmode.go`) now returns `PullRequestDecision{Halt: githubHaltReason(...)}` when `recordFound` and the failure is non-transient and non-token. The function's own doc comment was updated; this package-level twin was not.

**LOW** — `internal/landingshed/finalize.go:170`, `Finalize` doc
- **Summary:** it says "a no closes a recorded open pull request, drops the record and lands locally". It does not say that an unreadable rule read with a record is now a halt (Stuck) instead of a no.
- **Evidence:** the new `finalize_pr_test.go` row "AnUnreadableParentRuleWithARecordedPullRequestHaltsClosesNothingAndKeepsTheRecord" pins that halt. I did not see the `finalize.go` lines after 170, so this may be covered further down.

**LOW** — `PRReview` type doc, `prreview.go:28`
- **Summary:** "A no closes any pull request the run recorded and passes" no longer covers a recorded run whose rule read fails. That case now ends Stuck with Halt.
- **Evidence:** `decide` reads the record first and passes `recordFound` into `resolveWithParentRule`.

I found no other stale twins. The `docs/overview.md:606` line is a general summary and does not contradict the change. Both callers of `resolveWithParentRule` pass the new argument. The `judge` open-case change matches the closed case's `dirtyEndReason` verdict.

Tool calls:
1. `grep -rniE "reads as no rule|as no rule|rules for the parent branch" ... -l` over `.md` and `.go`: to find twin text outside the files in my context.
2. `grep -niE "as no rule|unreadable|rules for the parent branch"` in `doc.go`, `finalize.go` and `docs/overview.md`: to read the matching lines, which my context lacked.
