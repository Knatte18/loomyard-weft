---
round: 2
focus:
  - "internal/shellguard/rows.go publishCommandPattern and notWordPattern: settle whether a gh command word, subcommand word or -X/--method value spelled in double quotes (gh \"pr\" merge 3, gh pr \"merge\" 3, gh api -X \"POST\" repos/o/r/pulls/3/merge) is denied, and whether clustered short flags (gh pr review 3 -rb x, gh pr edit 3 -Bmain) and a mutation on a later line of a multi-line quoted gh api graphql query are denied or named as accepted passes in the publish row's guardrail comment."
  - "internal/shellguard/rows_integration_test.go TestRows_PatternsDenyAndAllow: settle whether each alternative of the publish pattern and each githubAllowed entry has a deny or allow row, and whether the change description's Testing claim matches the rows."
  - "internal/landingshed/prreview.go syncText and the wait loop, with internal/landingshed/prreview_wait_test.go TestPRReview_Wait: settle whether the failed-edit route (transient retry with the synced text unchanged, non-transient Stuck), the open-state and ReviewEnded gates, the zero-baseline branch and the warning re-arm are each pinned by a test row, and whether a persistently failing transient edit warns once per episode."
  - "githubAllowed in internal/shellguard/rows.go against the board entry's wanted gh set, and the comments at internal/shuttleengine/spec.go, internal/shellguard/rows.go and internal/shuttleengine/claudeengine/settings.go that describe what the publish deny refuses: settle whether the allowlist and the comments agree with the entry and with shellguard/doc.go."
---

The focus lists where round 1 found gaps, so the next review checks the fixes in the code as it stands, not the earlier report.
