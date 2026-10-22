# Darn review hard cap defaults to 2; orch stencil drops run_settings

Darn's template set `review_max_bounces: 1` but `review_hard_max_bounces: 4`, a hard cap far above its one-round default.
The orch role stencil worked around it by telling the orch to hand-write `run_settings` on every small task.
The default hard cap is now 2, and the stencil points small tasks at a profile instead.

## Changes

- `internal/darnengine/template.yaml`: `review_hard_max_bounces` goes from 4 to 2.
  With a one-round budget, a cap of 2 lets the soft-cap rule buy one more round for a design-wrong fix and no more.
  It still passes `Validate`, which requires the cap to be at least `review_max_bounces`.
- `internal/runprofile/template.yaml`: the seeded `light` profile gains `darn: {review_max_bounces: 1, review_hard_max_bounces: 2}` beside its loom values.
  This pins the profile's darn bounds should a hub raise its darn defaults.
- `contracts/stencils/orch/orch-template-role.md`: the small-task `run_settings` rule is replaced.
  A task that needs fewer review rounds than its recipe's defaults names a profile, such as `light`, in its board entry's `profile` field, and never gets hand-written `run_settings` for review bounds.
- `internal/darnengine/doc.go`, the `Validate` comment in `internal/darnengine/config.go` and `docs/overview.md` state the default as 2.
- Tests that pinned the old default follow the new value, and `TestLoad` in `internal/runprofile` now pins the `light` profile's darn block.

## Compatibility

A hub that relied on the old default of 4 now gets at most 2 rounds unless its `darn.yaml` sets a higher `review_hard_max_bounces`.
An existing hub's `profiles.yaml` is operator-owned and never rewritten, so only a newly seeded hub gets the `light` profile's darn values.

## To do by hand

The loomyard hub's own `review_hard_max_bounces: 2` in `darn.yaml` (on Bolt, outside this change) is now redundant.
Remove it from the prime with `lyx config darn` after this lands.
