# Named run profiles in one hub file, picked per task by a profile field

`run-profiles` · Task · enhancement, board, batten, config, loom · running · PR-Align

## Seen (2026-10-18)

- Each task that wants other review rounds than the hub's carries the numbers itself in `run_settings`, such as `{"loom":{"discussion_max_bounces":2,"plan_max_bounces":1,"webster_max_bounces":1,"merge_max_bounces":1}}` for a light loom.
- Batten's seeding copies a task's `run_settings` into the pair's `.lyx/config/<module>.local.yaml`, and the run reads that file from then on.
- No named set of run settings exists; the batcher's `cautious` profile is the batcher's own.

## Operator decisions (2026-10-18)

- A board entry gets its own field, `profile`, naming a run profile, beside `recipe`, not a key inside `run_settings`.
- The profile is read at seeding, when its values are written into the pair's `.local.yaml` with the task's own `run_settings` layered on top; a later change of the profile does not reach a run already seeded.
- Profile names, their settings and adding a profile must all be easy to do.
- Profiles live in one hub config file of their own, `profiles.yaml` on Bolt, as a map from profile name to settings in the same shape as `run_settings`, so one profile can cover several recipes:

  ```yaml
  light:
    loom: {discussion_max_bounces: 2, discussion_hard_max_bounces: 3, plan_max_bounces: 1, plan_hard_max_bounces: 2, webster_max_bounces: 1, merge_max_bounces: 1}
  budget:
    loom: {discussion_max_bounces: 2, discussion_hard_max_bounces: 3, plan_max_bounces: 2, plan_hard_max_bounces: 3, webster_max_bounces: 2, merge_max_bounces: 2}
    darn: {review_max_bounces: 1}
  docs:
    darn: {writer: "sonnet[medium]", review_max_bounces: 0}
  ```

- These are the starting profiles; Webster and Merge get no hard cap, since their `auto_accept` makes it unused.
- A profile may set any key a task's `run_settings` may set, hard caps, models and fans included.
- Editing:
  - `lyx config profiles` opens the file in the editor;
  - `lyx config profiles --layer default --set <name>='<json>'` adds or replaces a whole profile, so `--set` takes a JSON value for this file;
  - renaming a profile is adding it under the new name and removing the old.
- `lyx board profiles` prints the profile names and their settings, as `lyx board labels` does for labels.
- A profile's keys are checked against the recipe module's config as `run_settings` are today; an unknown profile name on a task is refused at the board write and at seeding.
- The standard of about 3-3-2 review rounds is no profile: it is the hub default a task without a profile gets; tuning the hub numbers waits for `review-setup-audit`.
