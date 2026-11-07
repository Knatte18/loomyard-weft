# Shared Claude settings only from heddle, and a HEDDLE.md overview

What lym sets in Claude Code's settings was split over two files.
Heddle renders `.claude/settings.local.json` in every hub worktree, and each lym session also gets a per-session `--settings` document.
Several settings that are the same for every session of a worktree lived in the per-session document.
After this change each setting has one home:

- **The worktree file** holds what every session of a worktree shares.
  This applies to sessions lym launches and to sessions the operator starts by hand.
- **The per-session document** holds what differs per session or role.
  This covers hooks that name the session's own paths, the standing denies, plugins and skills by preloaded skill, auto-memory by role, and the role's subagent effort.

Heddle also writes `HEDDLE.md` beside `heddle.json` in the Bolt checkout.
The file shows every entry heddle holds, grouped by target, so the operator can read in one place everything heddle sets.

## What changes

- **Moved out of the per-session document** (`internal/shuttleengine/claudeengine`, `settings.go` and `scope.go`).
  The following now come from the worktree file only:
  - `attribution`;
  - `promptSuggestionEnabled: false`;
  - `disableClaudeAiConnectors`, `disableBundledSkills` and `disableWorkflows`.

  The opened-session document loses `attribution` the same way.
- **New shipped heddle entries** in `internal/heddleengine/shipped.json`:
  - `lym-session-scope` carries the three `disable*` keys;
  - `lym-prompt-suggestion-off` carries `promptSuggestionEnabled: false`.

  The existing `lym-no-attribution` already carried `attribution`.
- **ListAgents.**
  The per-role `permissions.deny: ["ListAgents"]` for every session but the orch's is removed.
  The operator's rule is that every agent keeps the same tool set, so this difference goes.
  It goes by allowing the tool everywhere: the orch uses it, and the orch smoke cycle reads its address through `ListAgents`.
  The scope now denies no tool.
- **Effort defaults from `agents.yaml`.**
  Heddle derives a `lym-agent-defaults` entry (`internal/heddleengine/agentdefaults.go`, new layer `LayerDerived`) at each render and at `lym heddle show`.
  The entry sets `effortLevel` and `env.CLAUDE_CODE_SUBAGENT_EFFORT` to the efforts table's `default`, read from the `agents.yaml` the worktree resolves.
  An `agents.yaml` that does not load leaves the entry out, and the render warns.
  Per-role effort stays on the launch: `--effort` and the per-session `CLAUDE_CODE_SUBAGENT_EFFORT` both outrank the worktree file.
  The variable's name is now one constant, `agentconfig.SubagentEffortEnv`, which both packages use.
- **HEDDLE.md** (`internal/heddleengine/overview.go`, `RenderOverview`).
  The file holds a section per target, in table order, each with a sentence on where the target delivers.
  Each entry aimed at the target follows in render order with:
  - its slug;
  - where it comes from: shipped, shipped with the hub's override, derived from `agents.yaml`, or the hub's `heddle.json`;
  - its order and targets;
  - its text in a fenced block, a JSON-object entry's object re-indented as `json`.

  The file is rewritten whole and holds no state.
  The `lym heddle` write path (`set`, `remove`, override `set`/`remove`, `ChangeStore`) renders it under the Bolt write lock after each save, and the board's sync commits it with the store.
  Heddle's Bolt replayer now claims `HEDDLE.md` too.
  After each replay it renders the file afresh from origin's state, so two hubs' overviews never conflict.
- **Docs.**
  Updated the `heddleengine`, `heddlecli`, `claudeengine` and `agentconfig` package docs, the `heddle` CLI help, `claude.yaml`'s `session_scope.enabled` comment, `docs/overview.md` and `pattern/PATTERN-heddle-instructions.md`.
  The pattern gains the rules that a shared setting lives only in the worktree file, that effort defaults are derived and never copied, and that only `RenderOverview` writes `HEDDLE.md`.

## Design decisions

- **No model default.**
  The task asks for the machine-wide model and effort defaults from `agents.yaml`.
  `agents.yaml` as `agent-skills-and-effort` landed it holds efforts but no model, so only the effort default is rendered.
  A session started by hand keeps Claude Code's own model default.
  Adding a model key to `agents.yaml` is left to a follow-up.
- **`session_scope.enabled` governs less.**
  The switch in `claude.yaml` now covers plugins, skill overrides and auto-memory only.
  The connectors, bundled skills and Workflow tool are steered through heddle's `lym-session-scope` entry: a `lym_override` or a repository entry setting a key to `false`.
  No hub file value changes meaning in a way a reinterpretation could rewrite, so none ships.
- **Plugins and auto-memory stay per role.**
  `enabledPlugins` selects which skills a session lists, and plugins on this machine bring skills, hooks and agents but no tools.
  The same-tool-set rule therefore keeps that difference.
  Auto-memory is no tool either.
- **Lag on change.**
  A change to `agents.yaml` reaches a worktree's settings file at its next heddle render (a made worktree, a take-in of a re-render path, or `lym heddle render`).
  It reaches `HEDDLE.md` at the next heddle write or replay.
  A lym release that changes shipped entries likewise shows in `HEDDLE.md` only after the next write.

## Risks

- `promptSuggestionEnabled: false` is what lets `IdleSession` read an empty input box.
  It now reaches a lym session only through the worktree file.
  A session started where heddle rendered no file loses it: a worktree whose exclude list refuses the render, or a launch outside any hub.
  Such a session would show prompt suggestions, and its turn ends would not read as idle.
  The same now holds for `attribution`.

## To check by hand

- Not run: a live Claude Code session confirming that `effortLevel` in `.claude/settings.local.json` sets the effort of a hand-started session.
  Also confirm that Claude Code accepts every value the efforts vocabulary allows there (`xhigh`, `max`) without a settings error at startup.
- Not run: a live lym launch in a hub worktree confirming that the session still shows no prompt suggestion and that `IdleSession` passes with `promptSuggestionEnabled` coming from the worktree file alone.
- Not run: a non-orch role confirming that it now lists `ListAgents`, adding about 400 tokens of tool schema to its first request.
