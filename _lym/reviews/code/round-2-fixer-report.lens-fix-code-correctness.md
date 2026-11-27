```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (fix-code-correctness).

**LOW**: `Timeout: time.Duration(cfg.DescribeTimeoutMin+cfg.DescribeReviewTimeoutMin) * time.Minute` (`describereview.go`, `describeReviewSeat`), copied to the lead by `table.Seats[0].Timeout = reviewer.Timeout` (`describe.go`, `DescribeTable`). With `describe_timeout_min: 0`, both runs are now limited to the review timeout alone instead of the shuttle default.

- **Input:** `describe_timeout_min: 0`, `describe_review_timeout_min: 15`, reviewer configured.
- **Before the fix:** the reviewer's `Timeout` was 0, so `seatSpec` fell back to `table.Timeout`, also 0. `normalizeSpec` then used `shuttle.run_timeout_min` for both runs ("use 0 for the config default").
- **After the fix:** both seats get 15m, so the writer silently has 15 minutes in total.
- **Negative values:** `-5` was refused as a negative `spec.Timeout`; it now yields a positive 10m and runs.
- **Why it can happen:** `describe_timeout_min` is not in `LoadConfig`'s positive-value list (`config.go`), so nothing rejects either value. This is an edge setting; the template's 30 is unaffected.

I found nothing else in the changed code:

- **F1:** the constant equals the old literal at all three sites.
- **F2:** the new Warn assertions would fail against the old message. The `completeSummary` failure path is covered by the "fix what the cause names" wording.
- **F3:** for a configured reviewer, the lead and reviewer timeout assertions fail with the defect in place. An empty `describe_review` leaves the lead on the table's timeout. The chair-timeout reason in `multillm.go` reads `table.Chair().Timeout`, so it reports the new limit.
- **F4:** the new `ReplaceBeamDescription` row and all three integration assertions fail if the forwarding or check is dropped. The integration test ran and passed in tier1.

The gate runs shown in context did not include `internal/shedrecipe` or `internal/loomcli`, which consume `DescribeTable`. My grep found no assertion there on the Describe table's timeouts, but I did not run their tests.

Tool calls made (1 of 6):
- One `grep` for `DescribeTable(` and the timeout key and field outside `landingshed`, to find other callers or tests that depend on the old limits; the context showed only some of them.
