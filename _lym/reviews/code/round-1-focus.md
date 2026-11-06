---
round: 1
focus:
  - "internal/shuttleengine/claudeengine/scope.go and scope_test.go: does every session that gets the scope still write disableWorkflows, disableBundledSkills and disableClaudeAiConnectors in both the run path (Prepare) and the opened-session path (OpenLaunch), and is the omitempty zero-scope case (scope off, no hub) still free of the new key?"
  - "Goal item 1 requires checking that no session lym launches uses the Workflow tool: grep the repository (stencils, role prompts, hooks, engine code) for Workflow tool use, and settle whether the diff or commit message backs the claim in doc.go that none does."
  - "Goal item 2 asks to find a setting that works for a plugin's skills, or else say so, drop the dead skillOverrides entries and state what stays listed: does the diff drop exactly the plugin-derived entries, keep skills_off, and state what stays listed in doc.go, template.yaml and the ScopeConfig comment? Does the diff carry any claim about the allowlist or Skill deny alternatives that the commit or docs do not support?"
  - "Removal completeness: after dropping installedPlugin, lyPluginName and the skills-directory reading, is any reference left in code, tests, docs (docs/overview.md, doc.go, template.yaml, other docs mentioning the ly-plugin exception or per-plugin skill overrides), or the live claude.yaml defaults? Does readInstalledPlugins still handle a malformed plugins value and the sorted-id contract as before?"
  - "Goal Check: a skill the session preloads must still be available. Does the plugin-on rule (a plugin is on exactly when a preloaded skill is listed under its name) still hold after the refactor, including ids without '@' and the ly plugin, which no longer has a special case?"
  - "Test economy in scope_test.go: do the remaining and new tests cover the disableWorkflows behavior and the dropped overrides without duplicating each other, and was any removed coverage (plugin skill overrides, ly exception) the only test of a behavior that still exists?"
  - "Change description and launch proof: _lym/landing/summary.md was absent at seed time; if present by review time, check its claims (first-request tokens against 37,330, at most four launches, no headless launch) against the diff, and note that the diff itself carries no launch evidence."
---

Seeded from the decision record's Goal (a board-entry-only record) and the diff of the single commit over dbd67fed.
