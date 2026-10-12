# Fixer report — plan, round 2

## Changes

- F1 (fixed as proposed): card 9's Intent gains, beside the `docs/overview.md` sentence, a paragraph target for the battencli package comment at the head of `cli.go`, naming the overview verb, run visibility and line contents (D2), ordering, poll and backoff, the two-strike unreadable rule and the content-rows bound.
  No new target was added; `cli.go` holds the already-listed `internal/battencli#Command`.
- F2 (fixed as proposed): card 5 declares `plan:internal/statuspane#TestFormatMinutes` under Create and replaces the no-test sentence with a reference to it.
  Its Tests list gains the table-driven test in `pane_test.go` with exact-string rows for a sub-minute duration (`0m`), 12m, 59m, 60m (`1h00m`), `1h05m`, `23h59m`, 24h (`1d00h`) and `2d03h`.
  Card 5's budget line is now `internal/statuspane: 3`, card 6's `3 → 6`.
- F3 (fixed as proposed): card 1's `TestFixedHeightPinsMatchesRulesPlacedHeights` bullet gains `wantErr` rows for an unknown `Size.Kind`, `ShareNum` -1, `ShareNum` above `ShareDen` and `MinRows` 0, each on a below-parent strand, plus a non-error row where a hidden strand with a `ShareDen` 0 content size lays out and pins as the window without it.
- F4 (fixed as proposed): card 1's `TestPartitionByAnchor` row now reads "a hidden strand carrying a content size is dropped".

## Disputed

None.

## Deferred

None.
