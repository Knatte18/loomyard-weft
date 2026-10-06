---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/skillload.go: does ClassifySkillLoad count only main-chain entries after the latest load-message user entry, skip sidechain and malformed lines, and degrade to Verified false in each case card 1 lists, never reporting a skill unknown by absence from a missing skill_listing?"
  - "internal/shuttleengine/run.go loadSkillsThenPrompt and loadSkill: does the run offset move past every load turn end including the retry's, is the timeout skip without retry, the unverified path without retry, and the at-most-two-turns bound as card 2 states, and does a pane that dies in either turn end as a died startup?"
  - "internal/orchengine/watcher.go and state.go: does the reload step encoding (0 skills, retry sentinel, everything else pointer) read old persisted ReloadStep values and an empty ReloadRetry as the pointer step, are step and ReloadRetry written in one save, and can a restart before or after the move into the retry step re-send or double-retry?"
  - "Card 4 Verify: does `grep -rn 'SkillUnknown\\|SkillLoadSequence' --include=*.go internal` come back empty, and do the claudeengine session.go comments and doc.go match what card 4 says?"
  - "Warn vocabulary (shared decision warn-vocabulary): do shuttle and orch both log one Warn per skipped skill with keys skill and cause drawn from unknown, not loaded, timeout, and one `skill load unverified` Warn listing the skills?"
  - "Provider seam: does any message text, Skill-tool name or transcript shape appear outside claudeengine/skillload.go, in shuttleengine, orchengine or orchcli?"
  - "Per-card mechanical check: for the Delete group in card 4, did the type-specific mechanical check run against every listed target, not only the first?"
  - "Test economy: do the new startup_test.go, watcher_test.go and skillload_test.go rows extend existing tests as the cards say, and does any added test duplicate coverage of an existing one?"
  - "Doc comments in the changed Go files and doc.go files: do they follow the target repository's own comment conventions and CLAUDE.md rules, including one sentence per line?"
---

Seed focus only; it points at where to look and settles no verdict.
Review range: no rework round exists, so the range is the merge base with `main` to HEAD.
