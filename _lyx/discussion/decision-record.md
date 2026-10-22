# Decision record — darn's review hard cap defaults to 2

## Goal

A darn task gets a Darn-Review hard cap of 2 by default, so a small task needs no hand-written `run_settings` to keep its review short.

## Scope

- `internal/darnengine/template.yaml`: `review_hard_max_bounces` default goes from 4 to 2.
- `internal/runprofile/template.yaml`: the seeded `light` profile gains `darn: {review_max_bounces: 1, review_hard_max_bounces: 2}` beside its loom values.
- `contracts/stencils/orch/orch-template-role.md`: the small-task `run_settings` rule is replaced by a rule that a task needing fewer rounds than the default names a profile.
- The docs that state the darn default: `internal/darnengine/doc.go`, the `Validate` comment in `internal/darnengine/config.go`, and `docs/overview.md`.
- Tests that pin the darn template's hard cap default, if any, follow the new value.

Out of scope: the hub's own `review_hard_max_bounces: 2` in `_board/_lyx/config/darn.yaml`, which lives on Bolt outside this worktree and is removed from the prime with `lyx config darn` once this lands.

## Decisions

### The template default is 2

Decision: `review_hard_max_bounces` in darn's template is 2.
Rationale: the operator's call of 2026-10-22; with a one-round budget, a hard cap of 2 lets the soft-cap rule buy one more round for a design-wrong fix and no more.

### The seeded light profile carries darn values

Decision: the runprofile template's `light` profile carries `darn: {review_max_bounces: 1, review_hard_max_bounces: 2}`.
Rationale: the board entry asks for it, and it keeps the profile's darn bounds pinned should a hub raise its darn defaults.

### The orch names a profile, never hand-written run settings

Decision: the orch stencil's small-task bullet says a task that needs fewer review rounds than its recipe's defaults names a profile on its board entry, never hand-written `run_settings`.
Rationale: profiles are the one place run bounds are tuned; hand-written run settings drift per task.

### The hub's darn.yaml value stays out of this diff

Decision: the change leaves `_board/_lyx/config/darn.yaml` alone and names its removal for the parent.
Rationale: that file is hub config on Bolt, written from the prime through `lyx config darn`, never from a task worktree.

## Constraints

- `PATTERN-sparse-hub-base`: no hub base file is filled from the template.
- `PATTERN-comment-line-breaks`: Go comments keep one sentence per line.
- `review_hard_max_bounces` must stay at least `review_max_bounces`, whose default is 1, so 2 passes `Validate`.

## Auto-mode assumptions

- An existing hub's `profiles.yaml` is operator-owned and never rewritten, so only a newly seeded hub gets the light profile's darn values.
- Darn's `review_max_bounces` default stays 1.

## Open risks

- A hub that relied on the old default of 4 now gets 2 rounds at most unless its `darn.yaml` sets a higher cap.

## Acceptance criteria

- A darn config loaded with no hub or local value resolves `review_hard_max_bounces` to 2.
- A freshly seeded `profiles.yaml` holds the `light` profile's darn values 1 and 2.
- The orch role stencil no longer tells the orch to write `run_settings` for a small task.
- Every doc that states darn's hard cap default states 2.
