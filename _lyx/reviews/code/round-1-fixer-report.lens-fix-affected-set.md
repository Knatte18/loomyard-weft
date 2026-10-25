```yaml
lens: fix-affected-set
outcome: reported
```

The F6 fix did not cover every member of its set; the F3 and F4 sets are fully covered.

Task: rerun the search behind each set-premise finding (F3, F4, F6) and check the fix covers every member, not only the ones the review named.

**F3, median 4.1 s: covered.**
- Search: `4.1 s|median`.
- The only hit is line 53, now 3.9 s, so no twin remains.

**F4, under 1 ms: covered.**
- Search: `under 1 ms|< 1 ms`.
- No hits remain, and lines 77 and 129 now both say about 0.6 ms over a bare process start.

**F6, lines with two independent clauses: LOW.**
- Changed text: the F6 line-break edits, which broke only the eight lines the review listed.
- The same pattern is still unbroken on many other lines. Each has an independent clause after a `;`, or after a `, and` or `, but` or `, so`.
- Lines 5, 25, 26, 30, 31, 33, 49, 51, 57, 60, 62, 78, 79, 95, 97, 99, 100, 111, 112, 120, 121, 133, 137 and 139 are examples. My search was a regex, so it also matches some compound predicates that the rule exempts.
- Line 97 is the clearest case: `It also tried to answer the idle-path message; that call reached the real lyx, which refused it …`. It is the very line F5 re-indented, and it still has a `;` boundary.
- The finding's `fix` key named explicit line numbers, so the fixer carried it out as written. The premise is a set, though ("several lines hold two independent clauses"), and the set was larger than the named lines.
- Evidence: `grep -n -E ', (and|but|or|so) [a-z`]|; ' report.md` returns the lines above.

Tool calls:
1. `grep` for `4.1 s|under 1 ms|< 1 ms|median` in report.md, to rerun the F3 and F4 searches on the changed file.
2. `grep` for clause-joining `; ` and `, and|but|or|so` in report.md, to rerun the F6 search over the whole file. My context held the file only before the F6 edits.
