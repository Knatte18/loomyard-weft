# Round 3 fixer report — discussion

## Changes

- F1 (BLOCKING), fixed as proposed, with darn's `review_max_bounces` closed rather than floored.
  Premise checked: `Publish.Call` and `PRGate.Call` return Done when `RequirePRToBase` lacks the parent branch (`internal/landingshed/publish.go:118`, `internal/landingshed/prgate.go:56`), and `darn.review_max_bounces: 0` skips Darn-Review and Describe (`internal/darnengine/doc.go:3`).
  `verify_attempts` stays open: `NewDarn` refuses a value below 1 (`internal/loomrecipe/darnrecipe.go:29`).
  D4 now lists darn's `LocalKeys` as `writer`, `writer_timeout_min` and `verify_attempts` and landing's as `conflict`, `conflict_timeout_min`, `describe` and `describe_timeout_min`, with a reason per excluded key.
  D4's categories, its Bound and the `PATTERN-config-layers` Constraints line add what lands on the parent branch, reviews and approval gates.
  The acceptance row names `darn.review_max_bounces` and `landing.require_pr_to_base` among the refused keys.
  The floor-of-1 option is recorded under the support log's Rejected alternatives.
- F2 (MEDIUM), fixed as proposed.
  Premise checked: `configengine.load` expands markers through `envsource.Build(baseDir)` (`internal/configengine/config.go:168`), which reads `<anchor>/.env` before the process environment (`internal/envsource/envsource.go:29`).
  D4's Rationale now rests on the local layer adding no new untracked route and names `.env` as an existing one; R3 names the `.env` route and leaves it to the parent, and the support log's ledger gains Q12.
- F3 (MEDIUM), fixed as proposed.
  D6 states the strip compares unexpanded YAML nodes, a marker equal only to the same marker text; the acceptance row pins a kept literal equal to the walk's expansion of a marker below.
  The expanded comparison is recorded under Rejected alternatives.
- F4 (LOW), fixed with both halves of the proposal.
  D6 narrows its claim to keys the worktree file holds, states the additive case, and reports each such key as an `inherited` list beside `stripped`; D7's reconcile line and a new acceptance row name `inherited`.
- Support log: round 3 entry under Review rounds, and Q5 updated for the new categories.

## Disputed

None.

## Deferred

None.
