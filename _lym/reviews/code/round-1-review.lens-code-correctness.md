```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the committed diff, as one case per changed function or branch.

## List: function or branch, case run, verdict

- `agentDefaultsEntry`, valid `agents.yaml` (default `medium`): returns an entry with `effortLevel` and `env.CLAUDE_CODE_SUBAGENT_EFFORT`, both `medium`. Correct.
- `agentDefaultsEntry`, template only, no file: `LoadOrTemplate` resolves the template, which has `default`. Correct.
- `agentDefaultsEntry`, `agents.yaml` with no `default` (`invalidAgentsConfig`): `Load` fails, so an empty entry comes back with a warning. Correct.
  - The effect is below, finding 3.
- `Layers.addDerived`: it resets `Derived` and `DerivedWarning` each call, so a repeat call is idempotent. Correct.
- `Layers.Ordered`: the order is shipped, derived, repository, so a repository entry wins key collisions on the fold. Correct.
- `fileText`, `LayerRepository` condition (codex-file branch and the shipped/repository split):
  - A derived entry is aimed only at `claude-settings`, which is no file-text target, so the split never sees it.
  - Overridden shipped entries are still `LayerShipped`.
  - Correct.
- `Renderer.Render`: derived warnings reach the report. Correct.
- `Renderer.Show`, JSON target: derived entry folded in, warning dropped. Acceptable for a read-only show.
- `RenderOverview`:
  - File absent: it writes and returns true. Correct.
  - File equal: it returns false. Correct.
  - Read fails with something other than not-exist: it returns a wrapped error. Correct.
  - Write fails: it returns an error and leaves the old file, because `AtomicWrite` is atomic. Correct.
  - `ReadLayers` fails: error, nothing written. Correct.
  - Invalid `agents.yaml`: the derived entry and the warning are dropped silently. See finding 3.
- `overviewText`, empty target and empty entry text: a target no entry names gets "No entry.", and an empty text gives an empty fenced block. Correct.
- `entryBlock`:
  - Text containing a triple-backtick run gets a four-backtick fence. Correct.
  - JSON text that doesn't decode is shown as written. Correct.
  - Trailing newlines are trimmed. Correct.
- `targetReach`: the wording for prompt targets reads correctly with an engine, a role or neither. Correct.
- `Replayer.Claims`: matches only the root `HEDDLE.md` and `heddle.json`. Correct.
- `Replayer.Apply`:
  - Applied change: Save, then render, so the result is non-empty. Correct.
  - Nothing applied and the overview stands: Empty is true. Correct.
  - Nothing applied and the overview stale: it renders and returns non-empty. Idempotent on the second run, per the test row. Correct.
  - Render fails after Save succeeded: it returns an error with `heddle.json` already rewritten. This relies on fabric discarding a failed apply's dirty tree. I did not verify that. The scribble-then-fail fixture suggests fabric handles it.
  - A commit touching only `HEDDLE.md`: `Decode(nil)` gives an empty store, nothing applies, and it re-renders. Correct.
- `saveLocked` (and `ChangeStore` through it): saving then rendering is consistent because `ReadLayers` rereads the saved file. A render failure after a good save returns the "saved but not rendered" error. Correct.
  - The detached sync is not started on that path, so the saved file stays dirty until the next sync or the rerun the message names. This is a deliberate trade-off and is not a defect.
- `resolveSessionScope`:
  - Orch with no plugins and no skills gives a zero scope, which serializes as `{}` with the hooks. Correct.
  - Non-orch gives `autoMemoryEnabled:false` and no `permissions`. Correct.
- `settingsDoc` and `openSettingsDoc`: `attribution` and `promptSuggestionEnabled` are gone from the serialized documents. Correct.
- `subagentEffortEnv`: it uses the shared constant and the value is unchanged. Correct.

## Findings

1. **MEDIUM, design.** `internal/shuttleengine/claudeengine/settings.go` (the `settingsDoc` change), `internal/heddleengine/shipped.json` (the new `lym-prompt-suggestion-off` entry).
   - `promptSuggestionEnabled:false` and `attribution` now reach a session only through the heddle-rendered `.claude/settings.local.json`.
   - A worktree that no heddle render has covered since this release has neither entry. That includes a worktree made before the release, a standalone run outside a hub, or a worktree whose exclude list refuses the render.
   - In such a worktree `IdleSession` never reads an empty box, and commits and pull requests get Claude's credit again.
   - The writer's summary names this as a risk. The code does not guard against it.
   - The decision record asks for exactly this move, so it is a consequence of the record. It is still a correctness regression for those worktrees.
   - Evidence: `settings.go` diff hunk "settingsDoc"; `doc.go` text "a session started where none is rendered goes without them".

2. **LOW, design.** `internal/heddleengine/overview.go:37` (`RenderOverview`), compared with `render.go:95` (`Render`).
   - `RenderOverview` resolves `agents.yaml` with `Dirs{Board}` only. `Render` uses `Dirs{Anchor, Board}`.
   - `ConfigSpec` is `HubWide` and the `efforts` key is `CategoryNone`, so a worktree's local `agents.yaml` layer may hold `efforts`.
   - The `HEDDLE.md` default can then differ from the `effortLevel` that worktree's file actually gets.
   - Evidence: `internal/agentconfig/config.go:121-131`, `internal/configengine/layers.go:26-30`.

3. **LOW, design.** `internal/heddleengine/overview.go:37` and `render.go:95`.
   - `RenderOverview` drops `DerivedWarning`, so `HEDDLE.md` silently omits `lym-agent-defaults` while `agents.yaml` is invalid.
   - `Render` warns, but the derived entry then falls out of the fold. The next render strips `effortLevel` and the subagent-effort env variable from every worktree file, per the "takes out what that record holds" rule in `doc.go`.
   - A transient bad `agents.yaml` therefore removes the defaults from every worktree it renders into.
   - Evidence: `agentdefaults.go:190-194`; `overview.go:37`.

4. **Scope gap, not a code defect, for the handler.** Nothing in the diff renders a model default. `agents.yaml` has no model key, and the summary says so.

## Tool calls

1. `grep -n "func ConfigSpec"` in `internal/agentconfig/config.go`, plus `grep` for `Claims(` in `internal/fabricengine`. My context lacked whether `agents.yaml` is hub-wide and which layers it takes; the second grep only located the fabric replay claim site.
