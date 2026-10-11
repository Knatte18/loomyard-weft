# Plan-Review round 2 — fixer report

## Changes

- **F1** (BLOCKING, scope), fixed as proposed, in `_lyx/plan/10-discussion-marker-fresh-spawn.md`.
  Premise confirmed: `internal/testkit/envkit/envkit.go`'s `FullEnv` sets every loom seam (`DiscussionSpec`, `DiscussionTable`, `CommitDiscussion`), and `TestFullEnv_FillsEverySeam` asserts `NilSeams(FullEnv(t))` is empty.
  `internal/battencli/wire_seams_test.go`'s `intentionallyNil` table lists the loom-only discussion seams that batten's `wire` leaves nil.
  The card's `**Edit:**` list gains `internal/testkit/envkit#FullEnv` (a member glyph, since `envkit.go` is a non-test file and the Intent names that one symbol) and `internal/battencli/wire_seams_test.go`.
  The Intent has `FullEnv` fill `PrepareDiscussionSpawn` with a closure returning nil, beside `DiscussionSpec`, and has `intentionallyNil` gain `"PrepareDiscussionSpawn": "loom-only seam, batten has no discussion segment"`.
  The ImpactSummary now names every `Env` builder and seam table the field reaches.
  No other place needs the field: `internal/loomcli/wiring_seams_test.go` checks loom's `wire`, which card 10 already fills, and the `loomrecipe` and `shedbuild` fixtures build from `FullEnv`.
  `lyx loom validate-plan` returns `ok: true` with informational findings only.

## Disputed

None.

## Deferred

None.
