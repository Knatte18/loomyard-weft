```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Affected set per change, covered or not.**

1. **`codexengine.NewFromConfig` gains a `Seams` argument.**
   - Callers found: `internal/shuttleengines/shuttleengines.go:82` and `codexengine/fixture_test.go:149`. Both are updated.
   - The Claude `NewFromConfig` calls are a different function.
   - Covered.

2. **The standing items move from `prompt.md` to `developer_instructions`.**
   - Set found: the `Prepare` and guards doc comments in `codexengine`, `codexengine/doc.go`, the `OpenLaunch` comment, `docs/overview.md:714`, and the assertions in `prepare_test.go`.
   - All of these are updated.
   - No Codex-specific stencil text names the old prompt-file layout (grep found only Webster stencils, which are generic).
   - Covered, except finding F1 below.

3. **New `codex.yaml` keys `max_subagent_threads` and `plugin_hook_timeout`.**
   - Covered: `template.yaml`, `ConfigSpec` categories, `LoadConfig` validation, `TestLoadConfig_Template` and `TestLoadConfig_RefusesInvalidValues`.
   - Not covered: `docs/overview.md` (F1).

4. **New `agents-prompt` target.**
   - Covered: `target.go`, `promptTargetsFor`, `passagesOf`, `Show`, `heddlecli` help text, `doc.go`, `overviewIntro`, and `overviewPlaces`, which skips shared targets and lists their entries under each place.
   - Covered: the render and show tests, and the `PATTERN-heddle-instructions` entry that names the new reconcile render site.

5. **`codex-file` now carries `AGENTS.md` and the expanded `CLAUDE.md`, with an import record and a new `RenderRequest.Root`.**
   - Covered: `requestOf`, `heddlecli.renderInto`, `fileText` and its callers (`Show` passes the new return shape), the `render_test` fixture, and the `heddleengine` and `heddlecli` doc comments.
   - Not covered: `docs/overview.md` (F1).
   - `lymdirs.HeddleDirName` is the right directory for the record, per `PATTERN-lym-files`.

6. **`fabricengine.NotifyWorktreeMadeAt` and the hub reconcile render.**
   - Covered: `fabricengine/doc.go`, `hubreconcile/doc.go`, and the two `PATTERN-heddle-instructions` bullets.
   - Covered: the `fabricengine` tests that count made events (`requireCloneMadeEvent`, three events after the pair add) were adjusted, and the `hubreconcile` integration test was added.

7. **`shuttleengine.UserSetupSource` seam and the new `claudeimports` leaf.**
   - Covered: `PATTERN-leaf-packages` and `PATTERN-shuttle-provider-seam` both carry the declared entries, plus the package doc comments for `shuttleengine`, `shuttleengines`, `claudeengine`, `claudeimports` and `codexengine`.
   - The `claudeimports` import allowlist is enforced by its own `leaf_enforcement_test.go`.

8. **Declared PATTERN changes and commit trailers.**
   - The `PATTERN-heddle-instructions`, `PATTERN-leaf-packages` and `PATTERN-shuttle-provider-seam` entries all land in the diff.
   - No commit in the range carries a `Pattern-Departure:` trailer, so none needs judging.

9. **D7 shipped entries.**
   - `internal/heddleengine/shipped.json` already has `lym-batching` and `lym-plan-edits-before-gate` on `agents-file`, and no text in them names a Claude tool.
   - The move is pinned in a commit before the range (`22b74f24`).
   - Nothing is left in the diff to change.

**Findings**

**F1 — MEDIUM, consistency, `docs/overview.md:709-714`**
- **Summary:** The shuttle `codex` engine paragraph describes the engine's launch in detail (`sandbox_mode`, `writable_roots`, `skill_roots`, hooks, creel delivery). It does not mention any behaviour this diff adds.
- **Missing:** `developer_instructions` as the carrier of standing items, the 128 KiB refusal, the subagent effort and thread cap (`max_subagent_threads`), the user setup (global file, CLAUDE.md files above the root, skill listing, plugin session-start contexts with `plugin_hook_timeout`), the `project_doc_max_bytes` and `AGENTS.override.md` change, and the instruction-file pointers.
- **Evidence:** The diff touches that paragraph only to change "prompt file" to "developer instructions" in one clause. `grep -n "max_subagent\|plugin_hook\|developer_instructions\|AGENTS.override\|codex-file" docs/overview.md` returns nothing.
- **Rule:** The repository rule that docs for the kind of change land in the same diff.

No other member of the affected set is left wrong.

**Tool calls**
1. Bash: grep for callers of `NewFromConfig`, "prompt file" mentions, and `agents-file`/`agents-prompt` in markdown. My context lacked repo-wide callers and docs.
2. Bash: grep for the creel trust sentence users, the `PATTERN-shuttle-provider-seam` text, and "AGENTS.md else CLAUDE.md" wording. My context lacked other readers of these strings.
3. Bash: grep of `docs/overview.md` for the new keys and files, plus `max_concurrent_threads` and `claude-prompt`/`codex-prompt` users. My context lacked the doc's content.
4. Bash: grep for `RendersInto`/`DeliveryPrompt`/`DeliveryFile` users in `heddleengine` and `heddlecli`. My context lacked the other shared-target consumers.
5. Read: `internal/heddleengine/overview.go` lines 115-185, to check that the overview treats a shared prompt target correctly. My context lacked that file.
