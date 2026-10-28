# Codex hook delivery spike

`codex-hook-spike` · Task · research, codex, creel · running · Drift-Verify

## Goal

Find whether Codex can take creel posts through hooks the way Claude does, so a later task can move Codex sessions off the pane path.
Today Codex sessions keep the pane path (decision on `creel-hook-post`, operator, 2026-10-28), and Codex is used for review only at first.

## What is known

- `docs/research/creel-hook-channels-2026-10-28/report.md`: Codex `post_tool_use` with `additionalContext` and `systemMessage` delivers mid-turn on `gpt-6-luna` at `low`, and the operator sees it as `↳ Hook · [creel] #1 …`; it fires only after a tool call.
- `docs/research/creel-midturn-trust-2026-10-28/report.md`: on Claude, a `PostToolUse` block with `reason` is taken at once, and Haiku obeys only with a system-prompt sentence.
- `docs/research/codex-spike-2026-10-24/report.md`: its `Not tested` section leaves `codex exec` and `Tab` queueing open; folder trust is keyed on the git repository root.

## Cases

Codex CLI on `gpt-6-luna` at `low` effort, one run per case, no repeats: six runs in all.
Each case uses a hand-written hook script standing in for creel; no change to lym's code.

1. Idle wake: a hook comparable to Claude's `asyncRewake` that wakes an idle Codex session with a message, or a finding that none exists.
2. `Stop` block: a stop-time hook that keeps the turn going with the message as its reason.
3. `post_tool_use` block with a reason, beside the `additionalContext` channel already proven: is the message taken at the next step?
4. Trust without the sentence: the case 3 delivery with no sentence in the developer instructions, to see whether Codex acts on a `[creel]` scope change anyway.
5. Folder trust for a linked worktree: does the trust entry of a git worktree's root cover hook execution there, or does a linked worktree prompt again?
6. `Tab` queue and `codex exec`: whether a line queued with `Tab` mid-turn reaches the model at the next step, and whether `codex exec` runs the same hooks.

## Method

- Each session runs as a reed strand in this worktree, so the operator can watch with `lym reed attach`; each step waits about 20 seconds.
- A turn end is read from a hook writing a file, never by matching a prompt glyph in the pane.
- Each case's strand is removed with `lym reed remove --name <strand>` once its captures are written.
- A `CODEX_HOME` of its own for the spike, as the codex spike set up, so the operator's `~/.codex` stays untouched.

## Report

`docs/research/codex-hook-delivery-<date>/report.md`, per case: the hook config and script, whether the model took the message at its next step, later or not at all, quoting its words, a `tmux capture-pane -p` copy after delivery, and what the operator could see.
End with a table of case × outcome and whether it is visible to the operator.
