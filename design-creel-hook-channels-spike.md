# Spike: which hook channels deliver creel posts and show them to the operator, Claude and Codex

`creel-hook-channels-spike` · Task · research, creel, claude, codex · running · Drift-Baseline

## Goal

Find which Claude Code and Codex hook channels can deliver a creel post to an interactive session, and which of them the operator can see in the pane, before creel's delivery is redesigned around hooks.
A post need not look as if a human typed it; it must reach an interactive session, and the operator must see that it did (operator, 2026-10-28).

## What is known

- `PostToolUse` with `hookSpecificOutput.additionalContext` delivers mid-turn (task `creel-hook-delivery`), but the operator sees nothing, in the chat or in the expanded view (note `creel-hook-visible`).
- Claude Code's hooks docs (https://code.claude.com/docs/en/hooks, read 2026-10-28) state: `systemMessage` is "shown to the user" without saying where; `Stop` with `decision: "block"` and a `reason` keeps the turn going; a command hook with `asyncRewake: true` that exits 2 "wakes Claude immediately even when the session is idle", its stderr reaching Claude as a system reminder; `MessageDisplay` can return `displayContent`, shown on screen only.
- Codex CLI 0.164.0 lists `post_tool_use`, `pre_model_request` and other hook events and has an `additional_context_limit`; whether any of them delivers context to the model is untested (`docs/research/creel-hook-delivery-2026-10-25/report.md`, Codex section).

## Cases

Each case runs once, on one throwaway session, with a hand-written hook script standing in for creel (no change to lym's code):

1. Claude, `PostToolUse` returning `additionalContext` and a `systemMessage` naming the message, during a turn of several shell calls.
2. Claude, `Stop` returning `decision: "block"` with the message as `reason`, with and without a `systemMessage`.
3. Claude, an `asyncRewake` hook started at `Stop` that waits for a file and exits 2 with the message on stderr; the file is written while the session is idle, about 30 seconds after its turn ended.
   Also record what happens when the hook's timeout runs out with no file.
4. Claude, `MessageDisplay` returning `displayContent` that appends one creel line to the assistant's text.
5. Codex, `post_tool_use` returning additional context during a turn of several shell calls; if the model's next step shows no sign of it, repeat once with `pre_model_request`.

## Limits

- Claude cases on Haiku 5.5, Codex on its cheapest model at `low` effort, one run per case, no repeats beyond the one named in case 5.
- Each session runs as a reed strand in this task's worktree, so the operator can watch it with `lym reed attach`; each step waits about 20 seconds before the next so the operator can see it.

## Report

`docs/research/creel-hook-channels-<date>/report.md`, per case:
- the hook config and script used
- whether the model received the message, shown by its next step
- a `tmux capture-pane -p` text copy of the pane after delivery
- what the operator could see in the chat and in the expanded view
- for case 3, whether an idle session woke and started a turn by itself

End with a table of channel × engine: delivers to the model, visible to the operator, works when idle.
