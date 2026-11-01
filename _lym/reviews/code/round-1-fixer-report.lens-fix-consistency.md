```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F6 change set for sibling or twin inconsistencies (lens fix-consistency).

**Findings**

1. **LOW** — `tools/tokencount/main.go` package doc (the "With no slugs it counts the -last finished runs…" paragraph and the Recipes paragraph).
   - Summary: the doc does not mention the new `-last` refusal, and it does not say that a change description that fails to parse is listed under `No landed commit`.
   - Evidence: `run` now returns `-last %d counts no run; name at least 1, or give -since` when `-last` is below 1, with no slugs and no `-since`. `findLanding` now returns `Landing{Missing: noParsableDescription + ": " + err}`. The doc says only that "The section names each run without a landed commit and why", and gives no `-last` bound.
   - Why it matters: the doc paragraph that describes `-last` and `-since` is the twin of the new refusal and was left as it was.

2. **LOW** — `tools/tokencount/landed.go`, the `Missing` field of `Landing` (`noParsableDescription + ": " + err.Error()`).
   - Summary: the parse reason is a long, multi-part string, and `writeRecipes` prints it inside the parenthesised `No landed commit: slug (reason)` list.
   - Evidence: `summaryparser` errors such as `summaryparser: summary file archive/r/1:_lym/landing/summary.md: first non-blank line "..." is not a "# <title>" heading: ...` contain `%q` of the file's first line and parentheses. The other reasons are fixed constants.
   - Why it matters: the line stays on one line, because `%q` escapes newlines. It reads noisily, though, and nothing in the doc or the tests treats the reason as variable.

No other disagreements. These changes agree with their surroundings:
- Both wrapper headers match each other and match the report order in `main.go`'s doc.
- The `landed.go` doc comment for `findLanding` matches the new behaviour.
- The `main_test.go` header was updated.
- The `time_integration_test.go` header was updated.
- The `-since` header string in the test matches the format string at `main.go:295`.
- `ChangedLines`' doc comment is consistent with the `git show` change.

Tool calls: one `grep` of `internal/summaryparser/summary.go` for its error text, because the parse reason is embedded in the report line and my context lacked that text.
