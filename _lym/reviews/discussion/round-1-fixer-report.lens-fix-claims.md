```yaml
lens: fix-claims
outcome: reported
```

Task: fix-claims lens. I checked every bound, guarantee and code fact the fix added against HEAD.

**MEDIUM: the fix-here cap does not hold in the fallback cases**
- **Changed text:** the D4 Bound bullet "one episode fixes at most `pr_review_small_max` findings in place before the verb sends back". D2's Bound repeats it: "D4 caps PR-Review's in-place fixes at `pr_review_small_max` per episode".
- **Problem:** D4 sends every set to fix-here in three cases: the row declares no `on_reroute`, the target's entry gate does not hold, or the target perch approves without review (`zero_rounds`). In those cases the fix-here count grows but nothing ever sends back, so the cap is never reached.
- **Evidence:** D4's own fix-here list says "Fix here otherwise, and also when …". The record's Scope says recipes with no Code-Review segment keep fixing in place.

**MEDIUM: the parent is not guaranteed to review before merge**
- **Changed text:** the D4 Bound line "every fix lands as a commit on the open pull request, which the parent and the operator review before it merges".
- **Problem:** the row does not require a review by the parent or the operator before merge.
- **Evidence:**
  - `internal/landingshed/prreview.go:44` says "The wait ends when the pull request is merged or approved, or closed or continued by the operator."
  - `sessionStopWanted` (`prreview.go:686-697`) accepts any approval of the gated head.
  - `askParentReview` (`prreview.go:346-353`) asks the parent only when `pr_review_ask_parent` is on, and only once per record. Fixes made after that ask reach no new request.

**LOW: the parent's review exists only with `pr_review_ask_parent` on**
- **Changed text:** D2's Bound line "so the pull request body and the parent's review see every one".
- **Problem:** the pull request body does carry the regression section. "The parent's review" happens only when `pr_review_ask_parent` is on, and the sentence does not say so.
- **Evidence:** `prreview.go:346-353` returns without asking when the setting is off.

These claims hold:
- The send-back file sits in the loom scratch directory beside the Parent-Align fixer report (`internal/loomcli/subjects.go:283`).
- The row stops the session, re-reads the tree and resumes the session when the wake does not end the review (`prreview.go:657-674`). This backs "stops the session first and re-reads the tree, as it does for the continue flag" and "keeps waiting, with the session up".
- Today the rubric's `## Review input` reads an input only through the amendment note (`loom-rubric-code-review.md:139`).
- `zero_rounds` on Code-Bouncer makes it settle approved at once (`internal/loomrecipe/doc.go:10,89`).

Tool calls:
- Read `internal/landingshed/prreview.go:40-50, 280-300, 655-700`. My context had none of these spans: the wait's end conditions, the closed-pull-request session path, or the stop, re-read and resume logic of the wake.
