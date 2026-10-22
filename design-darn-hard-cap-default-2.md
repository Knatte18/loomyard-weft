# Darn's review hard cap defaults to 2, and the orch stencil drops hand-written small-task run settings

`darn-hard-cap-default-2` · Task · enhancement, config, stencil, orch · running · Darn-Bouncer

## Seen

Darn's template sets `review_max_bounces: 1` but `review_hard_max_bounces: 4`, a hard cap far above its one-round default.
The orch role stencil works around it: "A small task gets `run_settings` `{"<recipe>": {"review_max_bounces": 1, "review_hard_max_bounces": 2}}`", so small darn tasks carry hand-written run settings.
The loomyard hub now sets `review_hard_max_bounces: 2` in its own `darn.yaml` default layer, and profile `light` carries `darn: {review_max_bounces: 1, review_hard_max_bounces: 2}` (operator, 2026-10-22).

## Idea

Operator decision (2026-10-22): a hard cap of 4 is far too high when one round is the default; the hard cap by default is 2.

- Darn's template default for `review_hard_max_bounces` becomes 2.
- The template's `light` profile, if it ships one, carries the darn values 1 and 2 beside its loom values.
- The orch role stencil drops the small-task `run_settings` rule: a task that needs fewer rounds than the default names a profile, never hand-written run settings.
- Once the template default is 2, the loomyard hub's own `review_hard_max_bounces: 2` in `darn.yaml` is redundant and is removed.
