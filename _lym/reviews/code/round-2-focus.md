---
round: 2
focus:
  - "internal/shellguard/rows.go, the word-level exception scan (unquotedWordOtherThan, the end pattern that stops its run, and sessionBusCommandPattern): does the code keep the end pattern in step with every character the word unit excludes, and do the doc comments there and in internal/shellguard/doc.go let an author of another row build one that fails closed?"
  - "internal/shellguard/rows.go, sessionBusCommandPattern and the session-bus rows: do the patterns still deny and allow what the amended board entry expects, including at redirects, backticks, backslash escapes, a following command and the payload's end?"
  - "internal/shellguard/doc.go against PATTERN-code-conventions: does every paragraph the diff adds or changes document only internal/shellguard?"
  - "internal/shuttleengine/claudeengine/settings_test.go, the settings golden path, the -update flag and TestBuildSettings_MatchesGolden: is the path declared once, does the compare mode check as much as the test did at the review base, and does the update mode write what the compare mode reads?"
  - "_lym/landing/summary.md: do its claims, including the blast radius under '## Merge danger', match everything the diff changes and can break?"
---
