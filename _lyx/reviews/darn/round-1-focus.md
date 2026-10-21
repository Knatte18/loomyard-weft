---
round: 1
focus:
  - "internal/shellguard/rows.go `commandPositionPattern`: does the quote-aware scan still refuse real invocations (python, raw go, in-place edit, computed rm, machine sleep) after separators, subshells and `$(`/backtick forms, and is a command in a pipeline or after `&&` still reached?"
  - "internal/shellguard/rows.go `redirect-write` row and `notPrefixPattern`: does the allow list (`.scratch/`, `/tmp/`, `/dev/`, fd duplication like `2>&1`, variable-led targets) hold, and do `>`, `>>`, `>|`, `&>` on a heredoc operator's line deny as the board entry asks, with scratch writes still allowed?"
  - "internal/shuttleengine/claudeengine/settings.go `renderShellGuard`: does `shQuote` of each pattern keep the rendered hook command correct for patterns with a single quote, and is `testdata/settings-golden.json` consistent with the changed patterns and steer wording?"
  - "Repo rules: `internal/shellguard/doc.go` and any PATTERN.md invariant the new row or payload contract touches land in this diff; check the summary's claims about what each pattern lets through and falsely denies against the pattern comments."
  - "Test economy: do the new `rows_integration_test.go` rows duplicate each other or existing rows, and is any added behavior (for example the machine-sleep or computed-rm rows under the new scan, or `description`-field text) left uncovered?"
---

Seed focus from the board entry (issues #770 and #772) and the summary's account of the diff.
