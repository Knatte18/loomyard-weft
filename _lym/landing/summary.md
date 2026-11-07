# Render shared Claude settings through heddle, add HEDDLE.md overview

Every Claude setting that is the same for all sessions of a worktree now comes from heddle's `.claude/settings.local.json`.
The per-session `--settings` document built by `internal/shuttleengine/claudeengine` keeps only what differs per session or role.
Before, the shared settings were spread over both files.

## What changes

- Heddle's shipped layer gains two `claude-settings` entries.
  `lym-session-scope` turns off claude.ai connectors, bundled skills and the Workflow tool.
  `lym-prompt-suggestion-off` sets `promptSuggestionEnabled: false`.
- `claudeengine` no longer writes `attribution`, `promptSuggestionEnabled`, the connector, bundled-skill and workflow switches, or `permissions.deny`.
  Its documents now hold hooks that name one session's own paths and the per-role scope: plugins, skill overrides and auto-memory.
- The `ListAgents` deny is dropped.
  It differed by role, which breaks the rule that every agent keeps the same tool set, and no shared setting replaces it.
  Every session now has `ListAgents`.
- Heddle derives a `lym-agent-defaults` entry from the hub's `agents.yaml`.
  It sets `effortLevel` and the `CLAUDE_CODE_SUBAGENT_EFFORT` variable to the efforts table's default,
  so a session started by hand runs at the effort lym's own launches default to.
  Per-role effort stays on the launch.
  The entry follows the shipped ones at every render and in `lym heddle show`.
  A change to `agents.yaml` reaches a worktree's file at its next render.
  An `agents.yaml` that does not load gives a render warning and no entry.
- Every heddle write renders `HEDDLE.md` in the Bolt checkout beside `heddle.json`.
  It lists every entry, shipped, derived and the hub's, grouped by target in render order, with its source, order, targets and text.
  A JSON-object entry shows its object as a json block.
  `lym heddle set` and `lym heddle remove` render it under the Bolt write lock, and heddle's Bolt replayer claims it and renders it afresh after each replay.
  It holds no state of its own.

Closes the board task `claude-settings-model-effort`.

## Design decisions

- Only the effort default is rendered.
  `agents.yaml` holds no model key, and this change adds none.
  Lym sets each role's model at launch, and a session started by hand keeps the model the operator picks.
- The derived entry reads `agents.yaml` from the Bolt checkout and the worktree anchor.
  `HEDDLE.md` reads the Bolt checkout alone, so a machine-layer `agents.yaml` is not shown there.
- The Codex file target now takes only repository-layer entries aimed at the Claude file, and not derived ones, matching the shipped-layer behavior.

## To check

- The board entry asked for model and effort defaults, and the heddle file renders only effort.
  Check that dropping the model default is acceptable (`internal/heddleengine/agentdefaults.go`).
- Run `lym heddle set` on a scratch entry and confirm `HEDDLE.md` renders as expected.
- Open a hand-started Claude session in a hub worktree and confirm it picks up the effort default from `.claude/settings.local.json`.
