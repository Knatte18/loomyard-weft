---
round: 2
focus:
  - "internal/boardengine/taskguard.go `isRunAgentSession`: does the orch-role exemption accept a numbered orch thread such as `ly:some-task:orch-2` through `agentname.MatchesRole`, while a run agent role with a task slug is still refused?"
  - "internal/boardengine/taskguard_test.go `TestTaskCreationGuard`: do rows pin both a numbered orch thread and a slug-free non-orch name such as `ly:webster` as allowed to create a task?"
  - "plugins/ly/skills/board/SKILL.md: does the skill state that a session whose `LYX_STRAND_NAME` carries a task slug under a non-orch role cannot create or promote a task, and that numbered orch roles are exempt?"
---

Round 1's fixes touch the sites above; they are new text for a fresh reviewer.
