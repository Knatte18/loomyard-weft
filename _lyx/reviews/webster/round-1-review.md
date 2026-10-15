---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/shuttleengine/spec.go:65
    summary: "comments still name the retired config keys and fields: Spec.AllowAgentTool cites shuttle config claude_deny_agent_tool, and two claudeengine test comments cite ClaudeDenyAskUserQuestion and ClaudeDenyAgentTool"
    fix: "In internal/shuttleengine/spec.go line 65 replace the words whatever the shuttle config's claude_deny_agent_tool says with whatever claude.yaml's deny_agent_tool says. In internal/shuttleengine/claudeengine/settings_test.go line 218 replace ClaudeDenyAskUserQuestion with DenyAskUserQuestion, and in the TestBuildSettings_AgentAndBashHooks doc comment (line 418) replace ClaudeDenyAgentTool with DenyAgentTool. Comment text only."
  - id: F2
    severity: LOW
    class: decision
    location: PATTERN.md:14
    summary: card 2 trimmed clauses from five PATTERN entries no card touches to make room for the new entry, so the overview that agent prompts inline no longer states those rules
    fix: "Decide whether the trims stand. The background files still carry every trimmed rule, so the default is to keep them and record the choice in the commit message. If the rules belong in the overview, restore the clauses on the five entries (hub-suffix, config-layers, batten-bookend, parent-directive, spawn-color) and cut the same bytes from entries whose rule the diff itself touches, so the PATTERN headroom lint still passes."
---

### [LOW] Stale names of retired config keys in comments

**Location:** `internal/shuttleengine/spec.go:65`, `internal/shuttleengine/claudeengine/settings_test.go:218` and `:418`.
**Issue:** Card 6 moved the Agent and AskUserQuestion denies onto `claude.yaml`, and card 7 removed the shuttle config's Claude fields.
Three comments still name the old shuttle key `claude_deny_agent_tool` and the removed fields `ClaudeDenyAskUserQuestion` and `ClaudeDenyAgentTool`.
The `spec.go` one sits on an exported field's doc and now points a reader at a key `shuttle.yaml` retires.
Nothing breaks, since only comments carry them.

### [LOW] PATTERN overview entries trimmed outside any card

**Location:** `PATTERN.md` entries `PATTERN-hub-suffix`, `PATTERN-config-layers`, `PATTERN-batten-bookend`, `PATTERN-parent-directive` and `PATTERN-spawn-color`, changed in commit `16eefbba4` (card 2).
**Issue:** Card 2 asks for one new `PATTERN.md` line.
The commit also cut a trailing clause from each of five unrelated entries, such as "no stencil tells an agent to ask the operator" and "session shutdown precedes removal in one row".
I read the five background files, and each still states the cut rule, so no invariant is lost.
The overview is the part inlined into every spawned agent's prompt, so those agents no longer see the clauses.
The headroom lint explains the trim, but no card or decision records the choice.

## Verification

- Range: `status.json` gives `product.parent` `main`, and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, which is the eleven card commits.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` both pass.
- Cards 1-3: `TestBuildSettings_MatchesGolden` passes against the golden file written in card 1 and untouched afterwards, and it fails on a label missing in either direction.
  `renderShellGuard` produces today's two command shapes.
  `deniedRecoveryForms` is the operator list plus the table's `Forms`, and the self-check plants one row per form.
- Cards 4, 7, 8: `dropRetired` runs in `load` before `FillMissing`.
  `moveKeys` runs first in `ReconcileAll` at the worktree and local layers and in `ReconcileHubWideAt` at the board dir, and `moveRunSettings` runs first in `PrepareRunSettings`.
  `renameDenyPython` keeps value and comment and reports a dropped duplicate.
  Every reader uses `DenyPython`, and no `claude_deny_python` reader remains outside the migrate, the tests and the docs.
- Cards 5, 6: `LoadConfig` takes told `Dirs`.
  Only `Prepare`, `OpenLaunch` and `PromptCacheLifetime` call `loadConfigOrRefuse`, `New()` keeps its signature, and every `shuttleengines.New` caller passes the `Dirs` its own `shuttle.yaml` load uses.
- Cards 9, 10: a search of non-test code in `reedengine` and `reedcli` finds no Claude env key or namer choice.
  The only copy of the key names outside `claudeengine` is `tmuxkit.ServerEnviron`, which the plan keeps, pinned by `TestEnvScrub_MatchesTmuxkitServerEnviron`.
  Every production `AddSpec` built by shuttle sets `Provider` to the resolved engine name.
- Per-card mechanical checks: a search finds no remaining reference to any symbol the Delete groups of cards 2, 7, 9 and 10 name.
- Test economy: no new test duplicates an existing one, and every behavior the diff adds has a covering test.
  The new tests are plan-prescribed rows or functions, and the budget rows match.
- Comment conventions: the diff adds no end-of-line comment and no block comment inside a function body.
