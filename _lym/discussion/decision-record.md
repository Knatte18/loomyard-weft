## Goal

```markdown
## Rule (operator, 2026-11-22)

A per-task beam setting is found by one lookup, at call time:

1. The value set in the task's own worktree applies; a task sets it through `run_settings`, which batten copies into its worktree like every other setting.
2. A worktree that does not set it takes its parent worktree's value, and so on up through the beams to the prime.
3. When no worktree in the chain sets it, the hub's default layer applies, else the template's default.

"Sets" means written in that worktree's own file, not the merged value, so a hub default never counts as a worktree's own setting.
A value set lower down always overrides the one above, in both directions; absence means the parent's value.

## What to do

- Build that lookup once, in `dobbyengine` beside `beam.yaml`'s config, as the one reader for every per-task beam setting.
- Move `pull_request` onto it: the task's value in `run_settings` replaces the board field `pull_request`, and the modes that only decide how far the field counts, `required` and the field clause of `on` and `off`, go; `on`, `off` and `merge` stay as values.
- Move `land_notice` onto it: the task's value in `run_settings` replaces the board field `notify_on_land`, and the `per-task` mode goes; the values become `on` and `off`.
- Carry the existing board fields over into `run_settings` once, and ship the reinterpretation `PATTERN-config-layers` asks for each meaning change.
- `orch-autonomy-mode` builds `autonomy` on this lookup, so this task lands first.

## What was seen

`pull_request`, `land_notice` and the coming `autonomy` each wired a beam value and a task value its own way, so the same idea had three rules.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - beam-project-notify-on-land-dropped (class design, severity MEDIUM, open in rounds 1, 2)
  - carry-misses-runs-in-flight (class design, severity MEDIUM, open in rounds 1, 2)
  - duplicate-row-in-prepare-run-settings (class design, severity BLOCKING, open in rounds 2)
  - member-line-overrides-explicit-off (class decision, severity MEDIUM, open in rounds 1, 2)
  - per-task-keys-declared-at-several-sites (class design, severity BLOCKING, open in rounds 1, 2)
  - registration-test-lacks-keep-marker (class design, severity BLOCKING, open in rounds 2)
  - reinterpret-discovers-worktrees-and-anchors (class design, severity BLOCKING, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
