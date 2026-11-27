---
round: 2
focus:
  - "internal/shellguard/rows.go, the closed-heredoc pattern of the shell-edit row: run the rendered hook (grep -E over the JSON payload) on commands whose heredoc body holds a bare word alone on a line (Summary, done, a delimiter word other than the operator's own), followed by body or quoted text opening with sed -i, perl -i or tee into a source file. Settle whether the body is skipped as the board entry asks, with the operator word bare, single-quoted and double-quoted, and with <<-."
  - "Same pattern: settle whether tying a closing line to its operator's word lets any edit that is really outside a heredoc pass, or denies a non-editing command, including several heredocs in one command, two operators on one line, delimiter words holding - or . , empty bodies and an arithmetic << ."
  - "internal/shellguard/rows.go, the scopes read after a closed heredoc: settle whether shell -c scripts, single- and double-quoted, are read after a closed heredoc as they are without one, and what happens to a heredoc closed inside such a script."
  - "internal/shellguard/doc.go and the shell-edit row comment in rows.go: settle whether every stated reach, pass and false denial matches what the rendered hook does, and whether the package doc's reading rules agree with each other."
  - "internal/shellguard/rows_integration_test.go: settle whether the rows would fail on a plausible wrong pattern (closing word not matched to the operator, body repeat requiring a line, a missing scope), and whether any row duplicates another."
  - "internal/shuttle claudeengine testdata/settings-golden.json and Codex's rendering of the shell-edit row: settle whether the rendered hook command stays valid and acceptably sized and fast on long inputs after any growth of the pattern."
  - "_lym/landing/summary.md: settle whether its Merge danger section and design-decision claims match what the diff does."
---

The entries cover the match's reach in both directions, its tests, its stated limits and the change description.
