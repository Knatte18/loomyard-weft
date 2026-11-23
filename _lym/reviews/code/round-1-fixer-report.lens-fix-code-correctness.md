```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran each changed function by hand on a concrete input:

- **`layoutParagraph`:** it returns nil when a laid-out line reaches the limit and is not a trimmed written line. `reflowParagraph` returns the written lines in that case. `layoutParagraph` never returns an empty non-nil slice, so the nil check is sound.
- **`planEdit` and `collectEdits`:** the line shift counts only edits whose `line` is below the group's own, so a group's own edit is excluded. The result is 1-based `group.line + shift + index`, sorted.
- **Caller of the changed signatures:** `reflowSource` in the test helper, `main`, and `collectEdits` itself are the only callers, and all three were updated.
- **F1 row 1:** I traced the three-line paragraph at limit 120. The first cut is judged against the whole remainder and passes, the later written break cuts the second piece, and the final join then rejects the cut, giving a line of about 165 characters. That returns nil, so the source stays as written. Without the nil check the output would change.
- **F1 row 2 (limit 30):** the colon cut leaves the 38-character remainder with no further break. That line is not a written line, so the paragraph stays as written. Without the nil check it would split after the colon.
- **F4 row (limit 60):** the cuts at both colons are accepted. The piece through the second colon is 52 characters and the remainder is 25. A first-break version would leave 20 and 57 characters, so the row tells the two choices apart.
- **F3 test:** group A grows from two lines to three, so Bar's source line 9 becomes line 10. The list-item line is excluded by `IsProseLine`. Dropping the shift would give 9 and fail the test.

Residual notes, neither raising a finding:

- A group that `collectGroups` returned twice would now be reported twice. Its edits would already have overlapped, and I found no such duplicate.
- Prose lines inside a held-as-written paragraph are reported even when the line is unchanged. The review asks for exactly this.

Tool calls:
1. grep for other callers of `collectEdits`, `planEdit`, `layoutParagraph`, `longLines` and `reportLongLines`: no other Go file in the repo calls the changed `godocreflow` functions. The only extra matches are unrelated local variables named `planEdit` in `internal/websterengine` and `internal/weaverengine`. My context lacked a repo-wide search.
