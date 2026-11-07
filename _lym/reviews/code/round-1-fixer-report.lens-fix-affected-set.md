```yaml
lens: fix-affected-set
outcome: reported
```

Findings checked: F1 (rows using `wantFailedAfter`) and F2 (claims of an exact pin). No findings.

- F1: only two rows set `wantFailedAfter`, at lines 1809 (1h) and 1819 (1h + back-off). Both are covered by the new upper bound at line 1935. Every other failing row leaves it zero, so the `> 0` guard skips them as the review required.
- F2: the only "pins the doubling" claim in `_lym/landing/` is `summary.md:16`, and it is now reworded. No other file there says "exactly". The test file has no comment that claims an exact pin on `wantHeld`.

Tool calls (2 of 6):
1. `grep wantFailedAfter` in `startup_test.go`: to list every row and use of the field, which my context did not show.
2. `grep exactly|pins the doubling` in `_lym/landing/`, plus a `wantHeld` comment grep in `startup_test.go`: to find other overstated "exact pin" wording.
