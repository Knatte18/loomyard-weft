---
round: 2
focus:
  - "internal/landingshed/prreview.go, the PR-Review row's `judge` and `wait`, and the matching lines in internal/landingshed/doc.go and docs/overview.md: with the parent-review setting on, does the code do what the board entry says about the orch's answer, and do the docs state the same thing the code does?"
  - "internal/landingshed/prreview.go `askParentReview`: after `AskParent` posted successfully, can any path make the driver step the row again and post a second request to the orch, and does a test pin the outcome?"
  - "internal/loomcli/landingdeps.go `pullRequestDepsGuardedBy`: do the `ParentReviewWanted` and `AskParent` closures read the layer and notifier the setting `pr_review_ask_parent` is defined over, and does a test in internal/loomcli pin that?"
---

Round 1 found one gating design finding, so the focus targets the area it touched without presuming its outcome.
