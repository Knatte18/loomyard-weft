```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change the diff makes, I looked for the callers, tests, docs, stencils and fixtures it forces to change. The list below shows what I found and whether the diff covers it.

Sets per change:
- **`Spec.AllowAgentTool` removed.**
  - The `start.go` setter, `start_test.go` and the `claudeengine.go` call sites are covered.
  - The `buildSettings` and `buildDenyNotice` callers in the tests are updated, as are `doc.go` in `shuttleengine` and `claudeengine`.
  - A grep finds no remaining `AllowAgentTool`, `DenyAgentTool` or `steerAgent*` outside the retired-key plumbing. Covered.
- **`deny_agent_tool` removed from the claude config.**
  - Covered: `template.yaml`, the `Categories` entry, `configreg_test`'s `wantLocalDenied`, `configsync_test` and `config_test` (which gains a retired-key load row).
  - Covered: the `RetiredKeys` and `RetiredWayForward` declaration, which `configengine/layers.go` reads (`dropRetired` and `ValidateLocal`).
  - **Not covered:** the `KeyMoves` row in `configreg/keymove.go:20` (see F1).
- **Agent-deny prose.**
  - Covered: `docs/overview.md` (line 420), `claudeengine/doc.go`, `settings.go`'s header, `loomengine/plan.go`'s comment, `burlerengine/doc.go`, `shuttleengine/doc.go`, `shellguard/doc.go` and `pattern/PATTERN-interactive-spawn.md`.
  - A grep over `contracts`, `docs`, `pattern` and `internal` finds no leftover "nested agents", "Agent hook" or "no subagents" claim outside `docs/research`, which is a measurement record.
- **Headless-claude row.**
  - Covered: `shellguard/doc.go`, `claudeengine/doc.go` and the overview sentence.
  - Covered: the settings golden file, whose mask shrank from 64 to 16 combinations.
  - Covered: the `PreToolUse` entry counts in `prepare_test` (4 → 5 and 5 → 6) and `settings_test` (4 → 5).
  - I did not find a `PATTERN.md` entry that needs a separate edit.
- **Smoke tests.**
  - Covered: the `smoke_guardrail_test` rename and prompt.
  - Covered: the `smoke_cluster_test` deletion of the steer pin, with `orchcli/smoke_live_test` dropping its check for the deny text.
  - Covered: the `gatequiescence_test` header drops the reference to the Agent-deny pin test.
- **Stencils.**
  - The darn writer's step list is renumbered 2 to 6. A grep of the stencil finds no cross-reference to those step numbers (its only "Step 4" is the section heading).
  - No existing stencil rule is dropped or weakened: the old steps 2–5 survive as 3–6 with the same text.
  - The loom plan stencil gains a paragraph and loses nothing.
- **Declared PATTERN changes.** The rubric's constraints section declares no PATTERN entry changes. The `PATTERN-interactive-spawn` background file gains one bullet. Nothing is missing.
- **Commit messages and departures.** None of the four commits carries a `Pattern-Departure:` trailer, and none is needed.

Findings:

- **F1 — LOW, class `consistency`.**
  - Location: `internal/configreg/keymove.go:20`.
  - Summary: the key move `shuttle.claude_deny_agent_tool -> claude.deny_agent_tool` still targets a key that `claude.yaml` no longer has.
  - A reconcile of a layer still holding the old shuttle key writes `deny_agent_tool` into `claude.yaml` (`configsync/movekeys.go:411`, `placeValue`) and reports it as moved. The loader then drops it on every read (`configengine/layers.go:205`, `dropRetired`).
  - It is harmless to behaviour. But the move re-creates a retired key in a tracked config file, and the next local-layer validation refuses it with the delete-the-key way forward.
  - Evidence: `keymove.go:20`, `movekeys.go:392-411`, `claudeengine/config.go:63-64`.
  - Suggested fix: drop the move row, so the shuttle-side retired key (`internal/shuttleengine/config.go:84`) is simply left. Alternatively, leave it and state that it is intentional. The diff does neither.
  - The tests the move row touches (`configsync_test.go` around lines 2039–2180) would need their expectations updated.

I found no other member of the set that the diff leaves wrong.

Tool calls:
1. Grepped `contracts`, `docs`, `pattern` and `internal` for the old Agent-deny prose, because my context only held the diff.
2. Grepped `internal` for `claude_deny_agent_tool` and `deny_agent_tool`, to find the remaining readers and writers of the old key.
3. Grepped for `KeyMoves()` callers and `FromKey`/`ToKey` uses in `configsync`, to see how the retired key flows through reconcile.
4. Read `configsync/movekeys.go:335-419`, because the key-move write path was not in my context.
5. Grepped the darn stencil for step cross-references and the sources for `claude_deny_agent_tool`, to check the renumbering and the remaining key-move row.
