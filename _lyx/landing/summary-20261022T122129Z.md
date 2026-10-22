# Darn's review hard cap defaults to 2

Darn's template set `review_max_bounces: 1` but `review_hard_max_bounces: 4`, a hard cap far above its one-round default, so the orch role stencil told the orch to hand-write `run_settings` on every small task.
This change makes the default hard cap 2 and moves small-task tuning onto profiles.

- `internal/darnengine/template.yaml`: `review_hard_max_bounces` defaults to 2.
  With the one-round budget, the soft-cap rule can buy one more round for a design-wrong fix and no more.
- `internal/runprofile/template.yaml`: the seeded `light` profile carries `darn: {review_max_bounces: 1, review_hard_max_bounces: 2}` beside its loom values.
- `contracts/stencils/orch/orch-template-role.md`: the small-task `run_settings` rule is replaced by one saying a task that needs fewer review rounds than its recipe's defaults names a profile, such as `light`, in its board entry's `profile` field, never hand-written `run_settings`.
- `internal/darnengine/doc.go`, the `LoadConfig` comment and `docs/overview.md` state the new default.
- The darnengine and loomcli tests that pinned the template's hard cap of 4 now pin 2.

## Design decisions

- The `light` profile keeps its darn values even though they now equal darn's defaults, so the profile's bounds hold should a hub raise its own darn defaults.
- An existing hub's `profiles.yaml` is operator-owned and never rewritten, so only a newly seeded hub gets the `light` profile's darn values from the template.
- The loomyard hub's own `review_hard_max_bounces: 2` in `_board/_lyx/config/darn.yaml` is redundant once this lands, but that file is hub config on Bolt, outside the task worktree.
  It is removed from the prime with `lyx config darn` after landing, not in this diff.
