```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: checked that the fix covers every member of each set-premised finding.

**List (findings left unchecked):** F7, F10, F12, F19, F20. F10, F12 and F19 name single sites. F7 and F20 I judged from context only, with no search rerun.

**Findings**

1. **MEDIUM.** Changed text: `internal/hubreconcile/walk.go#carryMovedRunSettings` with `dobbyengine.RenameRunSettingsModes` (F3), and the `ReinterpretPullRequestModes` registration.
   - **Gap:** hub run profiles are a second store of `run_settings` blocks, and nothing renames or flags `beam.pull_request: on|merge` in them.
   - **Evidence:** `internal/configreg/runsettings.go#checkProfiles` loads each `<profile>.<recipe>` block through `PrepareRunSettings`, which runs `dobbyengine.CheckLocalValues` and so refuses `on` and `merge`. `internal/loomcli/sharedbootstrap.go` and `internal/battencli/runsettings.go` load profiles at seeding.
   - **Not covered by the fix:** `configsync`'s `moveBlocks` moves old key names only, never values. The carry calls `boardengine.RewriteRunSettings`, which visits board entries only, and the reinterpretation reads only `beam.yaml` files.
   - **Result:** a hub whose profile sets `beam.pull_request: merge` has that profile refused after the upgrade, with no notice. That is the failure F3 describes for board entries.
   - **This hub:** a grep of its `profiles.yaml` found no `pull_request`.

2. **LOW.** Changed text: `internal/agentconfig/reinterpret.go#DropPRReviewerRole` and its registration in `configreg.go` (F4).
   - **Gap:** only the hub base file is covered. A worktree's local `agents.yaml` holding `efforts: {pr-reviewer/opus: ...}` still makes `Load` refuse there, and the reinterpretation neither rewrites nor flags it.
   - **Evidence:** `efforts` is `CategoryNone` in `agentconfig.ConfigSpec`, so a local layer may set it. The rewrite ignores `worktrees`, and `RunsWithoutFile` is unset.
   - **Contrast:** the beam reinterpretations flag worktree files. The existing refusal does name a way forward, which is to remove the key from that layer.

3. **LOW.** Changed text: the new rows in `internal/loomengine/config_test.go` and `internal/configreg/runsettings_test.go` (F11).
   - **Gap:** F11's premise names "the retired `loom.pr` area or a flat `pr_review_*` name" in a local layer or `run_settings`. Both new rows cover only the `pr` area.
   - **Evidence:** the one flat-name row, "retired flat pr_review_small_max key ignored", writes the worktree file through `contents`, not a local layer or `run_settings`. The review's `fix` key asked only for the area.

**Sets found complete**
- **Renamed values:** no shipped template, doc or contract outside research still sets `pull_request: on|merge` or uses the old constants.
- **Session and verb leftovers (F13–F15):** the only remaining `pr_reviewer` and "PR-Review session" hits are the retired-key entry and its comment in `landingshed/config.go`. The remaining "parent-review ask" hits belong to the review rounds, which keep it.
- **"Keeps what it meant" (F5):** the remaining uses apply to `merge` and to the `land_notice` flags.

**Tool calls**
1. Grep for old `pull_request` values and constants, plus the hub's config dir: my context did not hold the repo-wide set.
2. Grep for `pr-reviewer`, session and removed-verb leftovers: only the named sites were in context.
3. Grep for `RunSettingsBlocks`/`MoveSettings` and "keeps what it meant": the other run-settings stores and meaning texts were not in context.
4. Grep for `runprofile.Load` callers and `pull_request` in the profiles files: the profile consumers were not in context.
