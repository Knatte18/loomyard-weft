---
round: 1
focus:
  - "Wanted 'Skills in config': does every role lym starts resolve its skills from `agents.yaml` (gate fixer included), and does any spawn site still pass a Go list or omit `ConfigRole`? Check each site the summary lists against `internal/agentconfig.Roles`."
  - "Load refusal in `internal/agentconfig/config.go`: is a role missing from the config refused at load, and do the hub-file fallback to the template's list and the Guard/None classing match the 'refusal at load, no role starts with no list' goal?"
  - "Skill order and dedup in `shuttleengine` Start: does the role list precede `Spec.Skills` with repeats dropped, and do callers that previously passed language skills still get them exactly once?"
  - "Wanted 'Subagents inherit': does the appended system-prompt passage in `internal/shuttleengine/claudeengine/subagents.go` give the parent its own list in one place, and do the changed stencils point at it rather than restate a list, without dropping or weakening an existing stencil rule?"
  - "Wanted 'Effort controlled': is an explicit `--effort` applied to every Claude launch and opened session whose model-spec names none, by role and model, with correct most-specific-key precedence and alias matching (`claude-haiku-5-5` to `haiku`)? Does the passage require `effort` on every Agent call, and none on a fork?"
  - "Backstop: the goal names `.claude/settings.local.json` rendered by heddle; the diff sets `CLAUDE_CODE_SUBAGENT_EFFORT` in lym's per-session settings instead. Judge whether this meets the 'set it as a backstop if one exists' item and whether the env value is correct for every launch path."
  - "Codex and non-Claude engines: does the change leave them working, with no Claude effort vocabulary reaching them?"
  - "Repository rules: `PATTERN.md` and `pattern/` changes, leaf-package allowlists for `agentconfig`, config-layer classification, and `docs/overview.md` land with the diff."
  - "Test economy: new tests across the migrated packages that repeat coverage already in `internal/agentconfig/config_test.go`, `shuttleengine/configrole_test.go` or `claudeengine/subagents_test.go`, and behavior added with no test."
  - "Summary claims: check that the stated behavior changes (fix review skills, verify fixer role move, orch reload list, `AppendSkills` removal) match the diff."
---

Decision record alone: the measuring stick is the `## Goal` entry (three bugs: skills in config, subagent inheritance, controlled effort).
The record's other sections are empty.
