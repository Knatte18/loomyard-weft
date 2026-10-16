# Fixer-report, webster round 1

## Changes

- F1: the three test literals naming the retired `fan_review` key now use `discussion_review`, as the review proposed.
  - `internal/seatengine/engine_test.go`: the `ModelKey` and the `wantText` of the cluster-fan reviewer row.
  - `internal/shedrecipe/entries_burlerhalves_test.go`: the `want` `ModelKey`.
  - `internal/burlerengine/review_test.go`: the `ReviewKey`.
- `lyx gate test` on the three packages passes.
- The remaining `fan_review` mentions are in `internal/loomengine`, where they test and implement the retirement itself.

## Disputed

None.

## Deferred

None.
