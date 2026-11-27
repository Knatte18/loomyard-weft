```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each set-shaped finding.

**Left unchecked:** none. F4 has a single member. I checked F1, F2, F3, F5 and F6.

**Findings**

1. **MEDIUM: F6 is not applied (`_lym/landing/summary.md`).**
   - The change set holds no edit to it, and commit 38c6e5b06 touches only `internal/`. My context shows it only in its pre-fix form.
   - It still says "**Blast radius:** shellguard".
   - It still says any bare-word line closes the body, although the listed words are now tied.
   - It still says "an in-place edit on a later body line after such a line is falsely denied", with no mention of prose.
   - Its Change section names `afterClosedHeredocScope`, which the fix removed. It also omits the `bash -c` reach after a heredoc, and the new passes (two operators on one line, digit- or dash-led words).
   - If the parent meant to edit it after the self-check, this is only outstanding work.

2. **LOW: a tab between `<<` and the delimiter word now passes, and nothing names it (`rows.go`, `closedHeredocPattern`, the F3 set of unrecognised closed heredocs).**
   - `cat <<\tEOF` with a real tab (also `<<-\tEOF`), then a body, `EOF` and `sed -i s/a/b/ f`, passes.
   - In the payload the tab is the two characters `\t`. The operator `<<-?[[:space:]]*('|\\"|\\\\)?` does not match them.
   - The `\\\\` branch needs two backslashes, the listed words must follow at once, and `notWordFrom`'s first position excludes `\`.
   - The previous pattern, `<<-?` + `shellLineUnitPattern*`, took `\t` as a unit (`\\[^"n]`), so the command was denied before this fix.
   - Neither the row comment nor `doc.go` lists this pass.
   - I derived this from the pattern text and did not run it.
   - Admitting `([[:space:]]|\\t)*` before the quote group would close it.

**Sets that are covered**
- **F2:** all three entries of `shellScopes` are rendered after a closed heredoc.
- **F5:** the reach statement at `doc.go:15` now carries the exception. The search found no statement of the heredoc reach outside the files already in context.

**Tool calls**
- One `git grep` for heredoc reach phrases outside `rows.go`, `doc.go`, tests, JSON and research docs. My context's earlier search was cut at 40 lines and did not cover those phrases across the whole tree.
