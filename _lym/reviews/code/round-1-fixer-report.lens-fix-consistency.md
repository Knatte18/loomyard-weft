```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix for F1 and F2 against the sibling text it touches (fix-consistency lens).

**Findings**

1. LOW. `internal/shuttleengine/startup_test.go:1739`, field `wantFailedAfter`, and the `wantHeld` comment at lines 1732-1734.
   - The `wantHeld` comment documents the new upper bound only for succeeding starts: "A start that succeeds takes less than wantHeld plus one apiErrorRetryBackoff".
   - `wantFailedAfter` has no comment. Its failure branch (lines 1932-1936) now also applies a one-back-off ceiling. That ceiling is skipped when the field is zero.
   - A reader of the struct cannot tell that a non-zero `wantFailedAfter` is now a window and not only a floor. The comment on `wantHeld` and the code agree, so this is a small gap.

2. LOW. `_lym/landing/summary.md:14-16` and `:20`, sibling bullets of the F2 edit.
   - Line 14 says the bound applies to "Every row whose start succeeds". The F1 fix added a bound on failing rows that have `wantFailedAfter > 0`, and the summary does not mention it.
   - Line 20 says the open turn after a hold fails "at least one hour plus one back-off". The row is now also capped at that value plus one back-off. This is a stale sibling, not a contradiction.
   - The F2 edit itself ("each to within one `apiErrorRetryBackoff` above its scheduled wait") agrees with the code for succeeding rows. Line 25 ("One shared upper bound") also agrees.

No other twin was left unchanged. The only comparison of `elapsed` with `wantFailedAfter` is in the failure branch. The two rows that set `wantFailedAfter` (lines 1809 and 1819) match the review's trace: the never-ends row fails at about 1h and the new row at 1h31s.

**Tool calls**
- grep for `wantFailedAfter`: to find every user of the field, since my context showed only the branch.
- Read of lines 1728-1745: to see the struct field comments, which my context lacked.
