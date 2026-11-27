---
round: 1
focus:
  - "internal/shellguard/rows.go closedHeredocPattern: which ordinary heredoc spellings does the match read, and which does it miss? Check delimiter words outside [A-Za-z0-9_], several heredocs on one line or in sequence, an empty body, a closing line that ends the command, and blank, indented or comment lines between the closing line and the edit."
  - "internal/shellguard/rows.go closedHeredocPattern: which commands that edit nothing are now denied? Check a body running past the real closing line to a later bare-word line, `<<<` here-strings, arithmetic `<<`, and `<<` inside quotes or comments. Compare each result with the entry's 'while still skipping the heredoc body'."
  - "internal/shellguard/rows.go afterClosedHeredocScope: it derives from shellScopes[0] alone. Settle whether a closed heredoc inside a substitution, subshell or shell -c script, followed by an in-place edit, is handled as the entry asks, and whether a command-position step after the closing line behaves as it does at the top level."
  - "Run the changed shell-edit pattern as the rendered hook runs it (grep -E over the JSON payload) against the entry's command shape and long multi-heredoc commands: check its size in internal/shuttle claudeengine testdata/settings-golden.json, any argument-length limit, and matching cost."
  - "internal/shellguard/rows_integration_test.go: do the new rows pin each behaviour the diff adds and each documented limit, and do the allow rows fail if the body were not skipped?"
  - "Check the claims in internal/shellguard/doc.go, the shell-edit row comment, _lym/landing/summary.md (reach, false denials, Merge danger blast radius) and the Root-Cause trailer against what the pattern does."
---

Seed focus from the board entry and the diff of the review range.
