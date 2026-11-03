---
round: 1
focus:
  - "internal/shellguard/rows.go and rows_test.go: does the per-engine steer and notice keep the plain text for an engine without its own entry, keep internal/shellguard standard-library only, and leave every Claude render byte-identical (D10)?"
  - "internal/shuttleengine/codexengine/guards.go: is every row of the shellguard table rendered, in table order, as a Codex PreToolUse hook matching Bash with the tested deny output, and does the deny notice announce every installed row (D10)?"
  - "codexengine guards tests: does one test pin that the rendered hooks cover every row, derived from the table rather than a literal count, as claudeengine's test does (D10)?"
  - "internal/shuttleengine/codexengine/skillload.go: does SkillLoader ask for each SKILL.md by absolute path in one turn, resolve `<plugin>:<name>` to the newest version under skill_roots, report an unknown skill and skip it, and classify the load turn from the rollout by the `skills/<name>/SKILL.md` tail, with unverified when the rollout cannot be read (D9)?"
  - "skillload.go version ordering and rollout parsing: failure paths for a missing root, a non-numeric or odd version directory, a truncated or unreadable rollout, and a command naming the tail in a different turn."
  - "codexengine.go and command.go: does the prompt file open with the deny notice, and does the launch line carry the hooks and the skill-load first turn as D6 specifies, without widening into other decisions' work (D6, D7, D8 belong to other parts)?"
  - "Scope: does the diff stay within D9 and D10, with no change that belongs to the project's other parts?"
  - "Test economy: do skillload_test.go, guards_run_test.go, guards_test.go and the command_test.go changes duplicate each other's coverage, or leave a new behavior uncovered?"
---

Seed focus for the P2 slice (D9 skills as files, D10 every shell guard row on Codex), set from the Sketch and the diff stat.
