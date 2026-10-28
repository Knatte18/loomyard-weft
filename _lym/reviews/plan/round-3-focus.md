---
round: 3
focus:
  - "Card 7 (`_lym/plan/07-test-tier-check.md`), the `test-tier-unstated` pass rule and its way-forward string: for a card that creates several test files at different tiers, or an `untagged` helper beside a tagged test, does the text a failing writer reads lead to a passing plan, and does the rule, spec row and test rows say how Intent lines are matched to a file when one line names more than one test file?"
  - "Card 7's D7 rows of `TestValidatePlan_CardChecks`: does each failing state the card pins (no tier line, several tags, `untagged` plus a tag, two files on one line) have a row whose Detail text names a fix that passes, and a row showing that fix passing?"
  - "Card 9 (`_lym/plan/09-doc-go-check.md`) and cards 7 and 8: for `internal/loomcli#loomCLI.validatePlanCmd`, whose Long help sentence do cards 7, 8 and 9 each rewrite, and is the ref typed `Edit` or `Extends` on each card to match whether existing content is changed?"
---

Both questions on card 7 settle one issue: whether a writer who hits `test-tier-unstated` can reach a passing plan from the failure text alone.
