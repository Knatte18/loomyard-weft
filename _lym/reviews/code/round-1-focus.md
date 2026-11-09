---
round: 1
focus:
  - "internal/reedengine/buttonclick.go decodeStoredAnswer and keptDollarEscape: settle whether the decode restores every typed answer exactly when tmux keeps the backslash, including a typed backslash directly before a $ (`\\$HOME`), a `$` followed by a digit, `-`, `?` or another non-letter, and a doubled `$$`."
  - "internal/reedengine/buttonclick.go decodeStoredAnswer: settle whether the marker check can misread an answer, for example a typed answer that itself begins with `$A` or `\\$A`, or a stored value on a tmux that unescapes every `$`, and whether the TrimPrefix branch keeps old-tmux behavior unchanged."
  - "internal/reedengine/buttonclick.go runButtonVia: settle whether the template string `\"\\$A%%%\"` is escaped correctly for tmux's command parser and whether the other metacharacters of the answer (quote, backslash, semicolon, tilde, `#{...}`) still reach the file intact, per the failing TestRunButton_RealTmux case in the Goal."
  - "internal/reedengine/buttonclick_test.go and doc.go: settle whether the new test rows each cover a distinct behavior of the decode, and whether the reedengine doc text matches what the code does."
---

The Goal asks for a diagnosis first; the diff took the code-fix branch, so review the fix for correctness against the failing real-tmux case.
