```yaml
lens: fix-code-correctness
outcome: reported
```

Task: the fix-code-correctness lens over the F26/F2/F5/F13/F27/F28 change set. One finding.

**MEDIUM — `CarryBeamTaskSettings` treats a run-settings `pull_request: off` as a retired off.**
- Changed text: `internal/dobbyengine/reinterpret.go`, the loop in `CarryBeamTaskSettings`. It has `if key == PullRequestKey && settings[key] == PullRequestOff { flagged = append(...); continue }`, applied to every key in `settings`.
- `boardengine.BeamSettingsByEntry` now puts a beam run setting into `Set` as well. Its own test row "the beam run settings carry as the entry's settings" shows `Set{"pull_request":"off"}` for a run setting that is explicitly `off`.
- Walk-through: a task has `run_settings.beam.pull_request: off`, and its worktree's own files set no `pull_request`. `Set` holds `off`, so `ownTaskSetting` reports `set=false` and the loop reaches the off branch.
- It flags "asked for pull_request off, which is not carried: under a required or merge parent it still opened a pull request". That message only fits a retired entry key.
- It then skips the write. `off` is a valid current value under the new lookup, so the task never gets it in its local layer.
- The separate `entry.RetiredOff` flag exists to tell the two cases apart, but the loop ignores it. The branch should apply only when `entry.RetiredOff` is set.
- No added test covers it. `TestCarryBeamTaskSettings` has only the retired-key `task-off` and a `task-seeded` with `land_notice`. A row with a run-settings `pull_request: off` and an unset worktree would fail with the defect in place.

Checked and found sound:
- The `Rewrite` signature change reaches all three call sites: `configreg`, `gateslot` plus its test, and the `hubreconcile` fake rewrite.
- `maps.Clone(entry.Set)` cannot be nil, because `legacyBeamSettings` always returns a map.
- The once-per-machine carry test and the unreadable-board test each fail without the marker logic.
- `ToldWorktrees` would fail with the old directory discovery.
- Early return for a project in `foldLegacyBeamFields` matches the updated store test.

Tool calls: none. My context already held every changed file.
