---
round: 1
focus:
  - "internal/configengine (layers.go, overlay.go, set.go, config.go) and internal/configcli: does every D4 site refuse an old name with both full names and its own way forward, match the old path exactly and only a non-mapping value for area-named keys, and can --unset remove old keys from a file holding several of them, including at the hub default layer?"
  - "internal/yamlengine/move.go and internal/configsync/configsync.go: does the move keep values and comments, drop null and both-present cases as D5 says, run before prune, sparseSeed and the profiles follow (D6), and stay idempotent on aliases, anchors, empty maps and non-mapping intermediates?"
  - "internal/hubreconcile/walk.go, internal/boardengine (runsettings.go, board.go) and internal/configreg/runsettings.go: does the run_settings carry (D7) keep unmoved keys, mark the walk skipped on any failure, roll back correctly, and are move lines mailed even when the walk fails after the hub-wide commit?"
  - "Every path that loads the loom or tapestry config outside lym config (the CLIs' pre-runs and arm functions, including webstercli, shuttleparity, battencli, beamcli, shedcli): does each run hubreconcile.EnsureOrWarn before the load (D5)?"
  - "Readers of the nested paths (internal/loomengine, internal/reviewbudget, internal/configreg/settinglookup.go, internal/dobbyengine, internal/shedrecipe, internal/runprofile/template.yaml, contracts/recipes): does every old key have a move-table entry matching D1, and does every default, fallback and category (D8, D9) keep its meaning?"
  - "Bare key names, per the parent's list in the decision record's last added entry (gatecli, batcher, battencli, beamcli, webstercli, hibernatecli, reportcli, preflightshed, weaverengine, sheddriver/tackler.go, shedarbiter, burlerengine/fix.go, loomshed/plancontext.go): does each message name every config key in full through configengine.KeyName and a named constant? Does the same hold for text the parent merge brought in, with shellguard's literal the one declared exception?"
  - "The merge commit 2d0729dd4 (git show --remerge-diff): are the conflicts in internal/dobbyengine/tasksetting.go and the claudeengine settings golden settled without losing either side, and did the merge bring in any loom or tapestry key or reference still in flat form?"
  - "Commit 57513a969 and the internal/shedverbs loop tests: is the cause of the shed step hang fixed, does every TestLoopIntegration_* test that spawns shed step have a seconds-scale bound that kills the child and reports stderr and the status file, and is every widened or removed assertion covered by a Loosened-Check trailer with a measured cause?"
  - "The six round-3 plan defects the parent's added entry orders fixed: is each built as that entry states?"
  - "PATTERN.md and pattern/ against the declared PATTERN changes, the move-table background under 'Keys the template lacks', and the temporary reachability allowlist entries named in the Loosened-Check trailers: did the entries land as declared and the allowlist lines come out?"
---

Seed focus for round 1, drawn from the decision record, its added entries, the plan overview and the commit log of the review range.
No change description exists under `_lym/landing/` at the time of seeding.
