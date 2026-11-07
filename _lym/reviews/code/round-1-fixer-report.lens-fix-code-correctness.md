```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 upper bound added to the failure branch of `TestStartup_LoadTurnAPIError`.

No findings.

- **Never-ends row (`wantFailedAfter` 1h, line 1809):** the limit is 1h + `apiErrorRetryBackoff`. The review traces this row's failure at about 1h, so it sits under the limit.
- **Open-turn-after-hold row (line 1819):** `wantFailedAfter` is 1h + `apiErrorRetryBackoff`, so the limit is 1h + 2×`apiErrorRetryBackoff`. The review traces the failure at 1h31s, under that limit. A deadline moved by twice the hold would exceed it, so the row now fails with that defect in place.
- **Rows with `wantFailedAfter` of 0:** the `tt.wantFailedAfter > 0` guard skips the check. This leaves the dying-pane and failed-send rows unbounded, as the review's fix key requires.
- **Other `wantFailedAfter` uses:** only these two rows set it. The change adds only an assertion and touches no signature, so no caller is affected.
- **Gate:** it passed at tier 0 (1309 passed, 0 failed).
- **Not run:** I did not re-run the traced elapsed values (about 1h and 1h31s). They rest on the reviewer's hand trace and the passing gate.

Further tool calls: 1. I ran `grep -n wantFailedAfter` on `startup_test.go` to list every row and use of the field. My context showed only lines 1900–1980 of the file, not the table rows.
