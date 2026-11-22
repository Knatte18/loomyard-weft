```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale siblings and twins left unchanged (fix-consistency lens).

**Findings**

1. **MEDIUM**: the `rememberRuns` and landing-notice file header in `internal/dobbyengine/landnotice.go` (lines 1-5) is stale after F8.
   - It says the notice goes "only when the run's land_notice setting read on while the run was ahead of Land".
   - The fix now reads `land_notice` again for a remembered run past Land that was never read (`unreadLandNoticeRuns`).
   - The `LandNoticeState.Member` comment, "Dobby first sees the run ahead of its Land row", still holds.

2. **MEDIUM**: the F22 twin was left in `checkTaskSetting` (`internal/dobbyengine/config.go`).
   - Its way forward still ends "in that worktree, or in the task's run_settings".
   - The review finding F22 says a `run_settings` change reaches a worktree only through Seed-Child, which `lym loom resume` does not run.
   - `finalize_pr.go` was fixed to say "a run_settings change applies from the next run", but the lookup refusal that `LoadConfig` and `CheckLocalValues` also use still offers `run_settings` as an immediate fix.
   - `CheckLocalValues` passes the same text for its upsert-time refusal, where `run_settings` is the right place.
   - The text should differ by caller, or say "from the next run".

3. **MEDIUM**: the Told-Geometry invariant is contradicted by the new code, and its wording was not updated.
   - `internal/dobbyengine/seam_enforcement_test.go` still says production code "takes every absolute path it operates on from its caller".
   - `reinterpret.go` now derives the hub as `filepath.Dir(worktreeRoot)`, the board as `hub/lymdirs.BoltDirName`, and the sibling worktrees by listing the hub.
   - The test passes only because the allowlist grew by `lymdirs`, `envsource`, `yamlengine` and `beamsetting`.
   - Either tell the geometry through the `Rewrite` arguments, or amend the invariant text for this one use.

4. **LOW**: the `Reinterpretation.Rewrite` doc in `internal/configreg/configreg.go` was not updated for `RunsWithoutFile`.
   - It still says Rewrite "receives the hub base file's bytes".
   - Rewrite can now receive nil bytes, and `Reinterpretation.ID` and the `hubreconcile` package doc still assume a file exists.

5. **LOW**: the `Load` doc comment in `internal/boardengine/store.go` (line 189) says a retired key "has it folded into its beam run settings".
   - Since F9 and F13, a `pull_request` of `off` and a project's `notify_on_land` are dropped, not folded.
   - `doc.go` and `foldLegacyBeamFields` were updated, but this twin was not.

**Calls made**
- One Bash grep over `.go`, `.md` and `.yaml` files, outside the files the fixer edited, for leftover `notify_on_land`, `parentPullRequestMode`, "parent's beam.yaml" and "run_settings, then run" wording. My context lacked a repo-wide search for stale cross-references. It found nothing else stale, and `parentPullRequestMode` has no remaining references.
