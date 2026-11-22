---
round: 2
focus:
  - "internal/dobbyengine/reinterpret.go, retiredMeaning and ReinterpretPerTaskSettings: does the rewrite of pull_request required to on tell the orch what a landing no longer does, and give a way forward, as PATTERN-config-layers' reinterpretation rule asks?"
  - "internal/landingshed/pullrequestmode.go, internal/beamcli/pr.go, internal/dobbyengine/config.go, internal/boardengine/store.go: is each per-task key (pull_request, land_notice, beam) and each value (on, off, merge, merge-in) declared once and referenced elsewhere?"
  - "internal/dobbyengine/reinterpret.go and tasksetting.go: does every read of a per-task beam setting go through the one lookup code, as the Layers and Per-task beam settings rules in pattern/PATTERN-config-layers.md require?"
  - "internal/hubgeom/tasksetting.go, TaskSettingChain: does it get another worktree's Location through lymcwd.NamedWorktree, as pattern/PATTERN-cwd-resolution.md requires?"
  - "internal/boardengine/store.go, foldLegacyBeamFields and ParseBoardTasks: does a legacy pull_request or notify_on_land reach a run already past Seed-Child, a replay input, and a beam project, and does a carried off keep its old meaning under a required or merge parent?"
  - "internal/dobbyengine/reinterpret.go and internal/hubreconcile/reinterpret.go: does the reinterpretation reach a beam worktree's own files, and does a failed read of a file it only flags abort the reconcile walk?"
  - "internal/configreg/configreg.go, beam module: is a beam run_settings value of pull_request or land_notice outside the allowed set refused at upsert and Seed-Child, and does a test pin the beam module's reinterpretation registration?"
  - "internal/dobbyengine/landnotice.go, rememberRuns: does a run whose land_notice read has never succeeded, or whose remembered state predates the upgrade, still get exactly one decision about its notice?"
---

Focus covers the gating design findings of the previous round; the reviewer still reads the whole range.
