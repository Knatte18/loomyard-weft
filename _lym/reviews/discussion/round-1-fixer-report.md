---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

# Round 1 fixer report — discussion

F4 is `decision`-class and went to the parent; no ruling reached this round.

## Changes

- F1: D4 now gives one way forward per site.
  A local or worktree file names `lym config reconcile --apply`, and a hub base file names `lym config reconcile --layer default --apply` from the prime worktree.
  `PrepareRunSettings`'s refusal names the old and new full names alone, so for a task's `run_settings` or a profile block only the caller's own way forward is printed.
  At Seed-Child that is the existing wrapper's `lym board upsert` or `lym config profiles` line.
  `lym config <module> --set` and a recipe reference name the new path.
  The D4 criterion now asserts the per-site way forward and one printed way forward at Seed-Child.
  The D5 bound, the D7 bound and the plan-writer note match it.
- F2: D6 is rewritten so that `configsync` moves `profiles.yaml`'s blocks, in the file and in the template base it reads, before `FollowTemplate`.
  This was the first option the fix offered.
  The block move uses the D7 helper under D5's new-path-wins rule, which no longer drops an operator value because the follow already compared at the new paths.
  The Constraints walk-order line and the Auto-mode assumption are updated to match, and a criterion now covers a changed shipped `solid.tapestry` block.
- F3: a failed D7 read or write now marks the walk skipped, so the build stamp stays stale and the next start verb retries.
  D7 runs no carry when the hub-wide write was skipped.
  The D7 criterion names the stale stamp and the next start verb's walk.
- F5: D5 now moves the old keys of the prime copy before `sparseSeed` prunes it, and a D5 criterion covers the seeded case.
- F6: the mixed-build risk is reworded.
  An older build refuses a hub base holding a map where the old key was a scalar, and otherwise warns and runs on template defaults.
  The scalar-turned-map keys are `discussion`, `raddle` and `friction` in loom, and `tapestry.yaml`'s `plan` and `parts` from D1's table are named beside them.
- F7: the D6 criterion now asks that the file parses to the same YAML value as the new template.
- Self-check: D4's closing sentence now limits the "only warns" claim to worktree and hub base files.
  For a local file, the old-name check replaces the existing unknown-key refusal and names `lym config reconcile --apply`.
  Support-log Q5 now says `configsync` moves the profile blocks and that every hub-wide walk carries board `run_settings`.

## Self-check

- fix-claims, MEDIUM: D4's closing sentence wrongly said a local file only warns on an unknown key. Fixed as proposed.
- fix-consistency, LOW: support-log Q5 still said a reinterpretation carries `profiles.yaml`. Fixed as proposed.
- fix-proposal: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
