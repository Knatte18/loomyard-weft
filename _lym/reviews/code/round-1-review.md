---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/configreg/keymove.go:20
    summary: the key move from shuttle claude_deny_agent_tool to claude deny_agent_tool stays, though its target key is retired and only the Agent deny ever used it
    fix: In internal/configreg/keymove.go delete the KeyMove row for FromKey claude_deny_agent_tool in KeyMoves; the shuttle spec already lists that key in RetiredKeys (internal/shuttleengine/config.go), so a layer holding it still loads. Then update the cases of TestReconcile_MovesKeysAcrossModules in internal/configsync/configsync_test.go that rely on the move, and add one row showing a shuttle layer holding claude_deny_agent_tool reconciles without writing deny_agent_tool into claude.yaml.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go:131
    summary: the headless-claude pattern misses a print flag bundled with another short flag such as claude -pc or claude -cp, and the row comment does not list it as a gap
    fix: In headlessClaudeCommandPattern replace the flag alternative (-p|--print) with (-[a-z]*p[a-z]*|--print), keeping the end-of-word class. Add deny rows "claude -pc hi" and "claude -cp hi" and an allow row for a single-dash word without p, such as "claude -c", to TestRows_PatternsDenyAndAllow in internal/shellguard/rows_integration_test.go. If bundled flags are meant to pass, instead add them to the guardrail-not-a-barrier list in the row comment of the headless-claude row.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/settings_test.go:378
    summary: TestBuildSettings_DenyToggleMatrix asserts no Agent PreToolUse entry, which TestBuildSettings_NoAgentHookAndForkGuard and TestBuildSettings_MatchesGolden already pin for every input
    fix: Delete the hasMatcher("Agent") check and its error message from TestBuildSettings_DenyToggleMatrix in internal/shuttleengine/claudeengine/settings_test.go, leaving the Agent absence to TestBuildSettings_NoAgentHookAndForkGuard.
---

### [LOW] Key move into the retired deny_agent_tool key remains

**Location:** `internal/configreg/keymove.go:20`
**Issue:** The decision record removes the Agent deny, its notices and the `deny_agent_tool` key "and whatever else only served them".
`KeyMoves` still moves `shuttle.claude_deny_agent_tool` to `claude.deny_agent_tool`, a key `claudeengine.ConfigSpec` now retires.
A reconcile of a layer holding the old shuttle key writes a dead key into `claude.yaml`, which the loader then drops on every read.
The change description calls this intended, but the row serves nothing: the shuttle spec already retires the old key, so a layer holding it loads without the move.
The `configsync_test.go` cases that used `deny_agent_tool` as the moved key were rewritten to `deny_ask_user_question`, so no test pins what the move does now.

### [LOW] Bundled short flags escape the headless-claude row

**Location:** `internal/shellguard/rows.go:131`
**Issue:** The pattern needs `-p` as a whole word, so `claude -pc hi` and `claude -cp hi` pass although they launch claude headless.
The focus names combined short flags among the forms the Goal's `-p` refusal should cover.
The row comment lists other gaps (`bash -c`, a script, an assignment prefix) but not this one, and the integration table has no row for it.
`env`, `sudo`, pipes and `&&` chains were run through the pattern and are refused as required.

### [NIT] Duplicate Agent-absence assertion

**Location:** `internal/shuttleengine/claudeengine/settings_test.go:378`
**Issue:** The deny-toggle matrix checks `hasMatcher("Agent")` per row.
`TestBuildSettings_NoAgentHookAndForkGuard` asserts the same absence over the fork and AskUserQuestion inputs, and `TestBuildSettings_MatchesGolden` pins the whole document for every input combination.

## Holistic check against the decision record

The Goal's five points are carried by the diff:
- subagents and forks are allowed in every run (both Agent denies and `Spec.AllowAgentTool` gone, the orch spec keeping `ForkSubagents`);
- a `headless-claude` shell-guard row refuses a Bash `claude` launch with `-p` or `--print` in every run, its steer and notice name a subagent through the Agent tool;
- the `deny_agent_tool` key is retired, so a layer holding it loads and a local override is refused with a way forward, and `steerWebsterForkDeny` stays;
- the darn writer and Plan writer stencils say fresh Sonnet subagent, per-package ownership, one message, the gate rule with its timeout, and what stays in the session, with no existing rule dropped;
- `docs/overview.md`, the package docs and `PATTERN-interactive-spawn.md` describe the new rule, and no leftover Agent-deny prose remains outside `docs/research`.

No commit carries a `Pattern-Departure:` trailer and none is needed, and the rubric's constraints section declares no PATTERN changes.
