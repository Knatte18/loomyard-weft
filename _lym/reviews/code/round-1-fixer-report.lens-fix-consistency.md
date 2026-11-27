```yaml
lens: fix-consistency
outcome: reported
```

Task: consistency check (fix-consistency lens) of the change set's agreement with its surroundings.

No BLOCKING or MEDIUM findings; three LOW.

1. LOW. `describeReview.releaseReviewer` and `describeReviewFiles` (the new `.ready` and `.sent` records in the review directory)
   - Two sibling comments still say the directory holds only the reviewer's report and end marker.
   - Evidence: `loomengine.DescribeReviewDir`'s doc says "the directory the Describe row's reviewer writes its report and end marker in … so neither file is committed". `DescribeInputs.ReviewDir` in `internal/landingshed/describe.go` says "the never-tracked directory the reviewer writes its report and end marker in". The gate now writes two more files there.

2. LOW. `describeReviewSeat`'s `Timeout: cfg.DescribeTimeoutMin`
   - `landing.describe_timeout_min` now also bounds the reviewer's run, but the key's own descriptions don't say so.
   - Evidence: `internal/landingshed/template.yaml` says "minutes the Describe session's shuttle run is allowed to run", and `Config.DescribeTimeoutMin` says "the Describe session's wall-clock budget". The sibling `describe_review_timeout_min` comments were rewritten and no longer name any bound on the reviewer's run.

3. LOW. `contracts/stencils/landing/landing-template-describe-review.md`, new `{{.ready_path}}` marker
   - The stencil's header comment says seatengine fills it "from the table's value map, which landingshed's DescribeTable builds".
   - `ready_path` comes from the seat's own values (`Values: map[string]string{"ready_path": files.ready}` in `describeReviewSeat`), not the table's map.

Tool calls:
1. Grep over docs, contracts and internal for statements about the operator-typed hold and the reviewer's budget or wait, since my context did not show repo-wide mentions.
2. Grep for PR-Review ending and text-compare wording and for places seeding the loosened-check fragment, since my context did not show those sites repo-wide.
