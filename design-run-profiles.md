# Named run profiles: a board field picks a central set of run settings

`run-profiles` · Note · enhancement, board, batten, loom, config

## Seen (2026-10-18)

- Each task that wants other review rounds than the hub's carries the numbers itself in `run_settings`, such as `{"loom":{"discussion_max_bounces":2,"plan_max_bounces":1,"webster_max_bounces":1,"merge_max_bounces":1}}` for a light loom.
- Batten's seeding copies a task's `run_settings` into the pair's `.lyx/config/<module>.local.yaml`, and the run reads that file from then on.
- No named set of run settings exists; the batcher's `cautious` profile is the batcher's own.

## Operator decisions (2026-10-18)

- A board entry gets its own field naming a run profile, beside `recipe`, not a key inside `run_settings`.
- The profile is read at seeding, when its values are written into the pair's `.local.yaml` with the task's own `run_settings`; a later change of the profile does not reach a run already seeded.

## Operator view, not a decision (2026-10-18)

- The standard profile is about 3-3-2 review rounds (Discussion, Plan, code perches).
- How many rounds each perch should have waits for an audit of the use of the twill and lens perches introduced on 2026-10-16/17, after they have run for a while.

## Orch recommendation, not a decision

- Profiles are defined once on Bolt, in the hub config of the module that owns the recipe (`loom.yaml`, `darn.yaml`), as a map from profile name to settings, such as `light` and `budget`.
- Names say what a profile is for, not its numbers (`light`, not `loom211`): numbers in a name say nothing about which perch they set, and the name breaks once a profile also sets models, fans or hard caps.
- A task's own `run_settings` override single values on top of its profile.
