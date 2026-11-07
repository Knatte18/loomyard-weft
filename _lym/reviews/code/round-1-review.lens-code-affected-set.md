```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change, the set I found and whether the diff covers it:

1. **The per-session `--settings` document drops `attribution`, `promptSuggestionEnabled`, the three `disable*` keys and `permissions.deny`.**
   - Covered: `claudeengine` doc.go, settings.go, scope.go, command.go comments, `template.yaml`, `docs/overview.md`, and the `scope_test`, `settings_test`, `command_test` and golden files.
   - Covered: the new shipped entries `lym-session-scope` and `lym-prompt-suggestion-off`, the `store_test` ordering row, and the `heddleengine` `settings_test` fixtures.
   - Searched for stale readers of the removed keys. Only the new test negatives and the heddle fixtures match, so nothing is left wrong.
   - The remaining `ListAgents` hits are the orch smoke test, which still works now that the tool is allowed everywhere, and historical `docs/research` reports. Those are records of past runs and are not stale.
2. **`SubagentEffortEnv` is now one constant.**
   - Covered: `subagents.go`, `subagents_test.go` and `agentdefaults.go` use it.
   - Not a gap: the literal `CLAUDE_CODE_SUBAGENT_EFFORT` survives only in doc comments and in the `subagents_test.go` expected-JSON assertion.
3. **The replayer claims `HEDDLE.md`, and `RenderOverview` runs after each replay and each `lym heddle` write.**
   - Covered: `heddlecli` cli.go, verbs.go and doc.go, the `heddleengine` doc.go, `docs/overview.md`, `PATTERN-heddle-instructions`, the replayer tests and the fabric replay integration test.
   - **Not covered:** the `cmd/lym/main.go` comment, finding 1 below.
4. **The `claude-settings` entry list gains a derived entry, and the codex file-target check changes from `Layer != Shipped` to `Layer == Repository`.**
   - Covered: `Show`, `Render` and the `fileText` callers.
5. **`session_scope.enabled` governs fewer settings.**
   - Covered: the `template.yaml` comment, scope.go and doc.go.
   - Reinterpretation: `PATTERN-config-layers` says a meaning change ships a `hubreconcile` reinterpretation. The change description says none ships, because no hub file value changes meaning. I judge that defensible, since a hub with `enabled: false` now gets the connectors, bundled skills and Workflow tool off from heddle, which is a behavior change, not a value rewrite. I raise no finding.
6. **PATTERN declared changes.** The directive's declared-changes list is empty, so nothing is owed. The `pattern/` diff is checked under finding 2 and reaches no other declaration.
7. **Commit messages.**
   - No `Pattern-Departure:` trailer appears, and none is needed.
   - The commit messages match the diff.
   - The commit message says `agentconfig.SubagentEffortEnv` names the variable once. That holds.

## Findings

1. **LOW, `consistency`: stale registration comment for heddle's replayer.**
   - Location: `cmd/lym/main.go:161`.
   - Summary: it says "A records replay merges heddle.json by entry through heddle's Bolt replayer." The replayer now also claims `HEDDLE.md` and renders it afresh after each replay. Its own sibling comment on the board replayer names what it claims.
   - Evidence: `cmd/lym/main.go:161-162` against `internal/heddleengine/replayer.go:24-26,108`.

2. **LOW, `consistency`: the `pattern/` edit removed the blank line before the next heading.**
   - Location: `pattern/PATTERN-heddle-instructions.md:36`.
   - Summary: the diff deleted the blank line that preceded `## Lym instructions in code`. The `HEDDLE.md` bullet now runs straight into the heading.
   - Evidence: the file at lines 34-37, and the diff hunk `@@ -28,8 +28,11 @@`, which removes the `-` blank line and adds none.

Tool calls (3 Bash):
1. Grepped for the removed settings keys, `CLAUDE_CODE_SUBAGENT_EFFORT` and `session_scope` outside tests, `_lym` and `docs/research`. My context lacked the callers and docs of these symbols.
2. Grepped `heddle.json` mentions in docs, Go code and fabric/board code, and ran `git diff --stat` on `PATTERN.md` and `pattern/`. My context lacked the rule documents and the Bolt-file listings.
3. Read `cmd/lym/main.go:155-168`, `heddlecli/doc.go` mentions and the fabric `BoltReplayer` definitions. My context lacked the registration comment.
