---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/settings.go and claudeengine.go: does the advisor reach the --settings document (or --advisor) only when the flag is set, leaving every launch without it byte-identical to today's?"
  - "internal/shuttlecli/run.go and internal/shuttleengine/spec.go: is the option per run only, with no lym config key added, and does the flag's value flow into the spec for every run mode?"
  - "internal/shuttleengine/codexengine: how does a codex run behave when an advisor is set (refuse before writing), and does the refusal leave no partial state?"
  - "Check-first requirement: do the option's help text and the shuttle module docs (docs/overview.md, claudeengine/doc.go, codexengine/doc.go) record the confirmed value form and tmux effect, and does the diff or commit log back that claim?"
  - "Scope: does the diff stay out of the advisor prompt, the bench, and a per-role config key?"
  - "Test economy: settings_test.go tests the redundancy report lists as contained (TestBuildSettings_CreelMailHook, TestBuildSettings_DenyToggleMatrix, TestBuildSettings_MatchesGolden); compare their assertions with the covering tests before filing anything."
  - "Commit log over the range: the trailing fix commits touch the Long text and doc comments; check that no rule of the advisor usage text was dropped along the way."
---

Seed focus from the decision record's board entry and the diff stat.
