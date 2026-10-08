---
round: 2
focus:
  - "Test economy in `_lyx/plan/06-burler-half-usage.md`: does the card plan any new test function for behaviour that `TestEngine_Run_ShuttleOutcomes` or `TestEngine_Run_ForkSubagentsSpecWiring` in `internal/burlerengine/engine_test.go` already covers, instead of extending them?"
  - "Test economy in `_lyx/plan/08-accumulated-exclusions.md`: is any one rule (union of focus-file excludes, eligibility from the previous round's usage record) pinned by two tests at different levels, such as a `burler_test.go` producer table and a `focus_test.go` direct test of `excludedLenses`?"
  - "Test economy in `_lyx/plan/05-claude-session-usage.md`: do the planned `usage_test.go` cases of `Claude.SessionUsage` read as rows of one table-driven test, or as several test functions for one behaviour?"
  - "Any edit made to a card's test plan since the plan was last reviewed: does the new text keep each test at a module's public surface and covering something no existing test covers, per `PATTERN-test-economy`?"
---

Focus targets the cards whose test plans changed in response to round 1.
