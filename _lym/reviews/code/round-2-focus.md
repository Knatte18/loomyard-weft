---
round: 2
focus:
  - "internal/shuttleengine/codexengine/codexengine.go Prepare: check that the hub gate config's refuse_raw list reaches the launch hooks and the prompt file's deny notice, that a present gate.yaml that fails to load refuses the launch before any file is written, and that a test drives each path."
  - "internal/shuttleengine/codexengine/guards.go and guards_test.go: check that the deny JSON output and the Bash matcher are asserted as literals, not rebuilt from denyDecision, hookEntry or toolMatcherBash."
  - "internal/shuttleengine/codexengine/skillload.go skillPath, compareVersions and ClassifySkillLoad: check version ordering is a total order, skill roots holding glob metacharacters still resolve, and the load line matches a rollout message even when the plugin cache changed after launch."
  - "docs/overview.md shuttle entry for the codex engine: check it matches what the diff adds for skill loading and shell guards while keeping the fork audit and creel hooks claims accurate."
---

Round 2 re-reviews the committed diff after the round 1 fixes.
