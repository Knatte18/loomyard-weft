---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/battenshed/innerrun_awaiting_test.go:361
    summary: The driver-busy notice's new leading state text is asserted by no test
  - id: F2
    severity: LOW
    class: design
    location: internal/battenshed/notice.go:158
    summary: pairRelative strips the pair prefix wherever it occurs in the text, including inside a longer absolute path that merely contains the pair directory
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, `product.parent` is `main`.
The diff makes the four changes the board entry asks for: no attach clause, pair-relative paths, `error:` only for failed and blocked with `waiting:` otherwise, and a condition phrased by what happened.
No remaining use of `AttachDir` exists in code, tests, docs or `envkit`, and the rename stays within what the entry licenses.
No doc or testdata still quotes `child left running` as a notice line or the attach clause; the `doc.go` phrase "a child that left running" is descriptive prose, not a notice quote.
An empty `Status.Error` omits the reason clause, and `TestRenderNotice` pins that for a done child.
A path already relative or outside the pair is left untouched, since only the `pairDir` + separator prefix is removed.

### [LOW] Driver-busy notice's leading state text is uncovered

**Location:** `internal/battenshed/innerrun_awaiting_test.go:361`
**Issue:** `noticeDriverBusy` now builds its condition as `haltText(w.status) + "; " + noticeDriverBusyCondition`, and `_lyx/landing/summary.md` claims it leads with the state text.
The only test reaching it filters on `driver pane busy` and never asserts the line's leading state text or the absence of the attach clause and absolute paths.
`TestRenderNotice` renders only `haltText` through hand-built conditions, so it does not cover this call site.
**Fix:** Add one assertion to the existing busy-notice test, on the collected `busyNotices` line, that it contains `awaiting at ` and `driver pane busy` and not `/wt/task`; no new test is needed.

### [LOW] pairRelative also strips the prefix inside a longer path

**Location:** `internal/battenshed/notice.go:158`
**Issue:** `strings.ReplaceAll(text, pairDir+"/", "")` removes the prefix at any position, so an absolute path outside the pair that contains the pair path as an interior segment, such as `/other/wt/task/x`, is rewritten to `/otherx`.
The case is rare, because a pair path is long and rooted at the hub.
**Fix:** Strip the prefix only where it starts a path token, meaning at the start of the text or after a space or a quote or colon-space delimiter, or accept the approximation and say so in the doc comment.
