---
round: 2
focus:
  - "In internal/landingshed/prreview_test.go, check whether TestPRReview_Call has a row with an empty credit block (Credit off) that asserts the Create or Edit call carries the trimmed description body alone, with no trailing blank line or credit lines; settle whether the body-alone branch of bodyWithCredit in internal/landingshed is covered."
  - "In internal/landingshed/prreview.go and prreview_test.go, check whether a PR-Review test makes EnginesRun fail and asserts the call returns the wrapped error before any FindOpen, Create or Edit call; settle whether the engines-run read-failure path is covered."
  - "In internal/summaryparser/summary.go and internal/loomcli/prcontinue.go, check that LandingMessage's parameter name and doc, and the prcontinue.go file header, describe the credit block as plain lines and name no git trailer, consistent with the surrounding docs."
---

Focus targets the sites of the round 1 findings; the review still covers the whole range.
