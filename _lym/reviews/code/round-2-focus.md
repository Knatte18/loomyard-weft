---
round: 2
focus:
  - "internal/gatecli/scopes_integration_test.go, the row 'a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor': is the commit its `--all` run records as whole-green different from `f.mergeBase`, so that the derivation assertion fails if a dirty tree diffs from that ancestor, and are the row's other steps and assertions as strong as at the review base?"
  - "_lym/landing/summary.md, `## Design decisions`, the bullet 'The plan is written for every scopes row': does what it says about where the plan sits and why no row's diff sees it agree with `planparser.PlanDir` and the pair's `.git/info/exclude`?"
  - "internal/gatecli/test_integration_test.go, the `scenarioHub` doc comment: does what it says about which rows build a hub, share state and run in parallel agree with the rows of the tests that use it?"
---
