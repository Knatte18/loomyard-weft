---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/scope.go: does every role's allowlist match the Goal: ly plugin on for the orch and board session in every hub and off for writers, reviewers, fixers, Webster and drivers; scribe skills limited to those the stencil and language.yaml preload; prowler off unless a role uses it?"
  - "scope.go and settings.go: does the written settings cover each lever the Goal names (disableClaudeAiConnectors, skillOverrides for anthropic-skills, disableBundledSkills, autoMemoryEnabled false for spawned roles only) and does it override the user settings and the repository's .claude/settings.json?"
  - "command.go: is --disallowedTools ListAgents passed to spawned roles and not to the orch or other interactive sessions?"
  - "Interactive versus autonomous split: the commit message says an interactive session keeps all its plugins, memory and ListAgents. Does the Goal license that, given it says memory stays on for the orch only and the other levers apply to every session?"
  - "Commit 52e60088 adds scribe:prose and scribe:conversation to the board button's preloads. Does the decision record license this change, and does it keep the board session's scope consistent with that preload?"
  - "reedengine templates (template_posix.yaml, template_windows.yaml) and claudeengine template.yaml, config.go: are the new session_scope and skills_off keys read, defaulted and validated on every path, including an absent base config? Is the reedengine edit within scope?"
  - "Coverage: does scope_test.go check the written settings per role, as the Goal's last bullet asks, and does each behavior the diff adds have a test? Check that no test duplicates one that already covers the same behavior, and that none asserts static prose."
  - "Per the Added after Discussion entry, the live launch check and start-load measurement are out of scope. Does the diff add any such trial, script or measurement?"
  - "docs/overview.md and claudeengine/doc.go: do the doc changes match what the code now does? Check the summary claims in the commit message against the diff."
---

Seed focus drawn from the decision record's Goal, its one Added-after-Discussion entry and the diff stat.
No finding is pre-judged.
