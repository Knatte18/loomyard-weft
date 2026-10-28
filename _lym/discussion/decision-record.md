## Goal

Find which Claude Code and Codex hook channels can deliver a creel post to an interactive session, and which of them the operator can see in the pane.
The result is a research report at `docs/research/creel-hook-channels-<date>/report.md` that a later task redesigns creel's delivery from.

## Scope

- In: five cases, each run once on one throwaway session with a hand-written hook script standing in for creel — Claude `PostToolUse` with `additionalContext` and `systemMessage`; Claude `Stop` block with and without `systemMessage`; Claude `asyncRewake` at `Stop` including the timeout with no file; Claude `MessageDisplay` with `displayContent`; Codex `post_tool_use`, then `pre_model_request` only if the model shows no sign of the context.
- In: the report with the hook config and script, the model's next step, a `tmux capture-pane -p` copy, the operator's view in chat and expanded view, the idle-wake result for case 3, and the closing channel × engine table.
- Out: any change to lym's code, creel, plugins or the operator's settings.
- The board entry's body and scope are not touched.

## Decisions

- Hook scripts, settings files and run logs live under `.scratch/`, and the committed report carries the scripts and configs it needs inline.
  Rationale: the board entry forbids changes to lym's code, and `.scratch/` is the sanctioned scratch location.
- Each session is a reed strand in this worktree, launched with `lym reed add --cmd`, with a private settings file via `--settings` and `--setting-sources project` so the operator's settings are not read.
  Rationale: the operator watches with `lym reed attach`, and the 2026-10-25 test shows this isolation works.
- Claude runs on Haiku 5.5 and Codex on its cheapest model at `low` effort, one run per case; the one allowed repeat is case 5's `pre_model_request`.
  Rationale: the board entry's limits.
- "Operator can see it" is judged from the pane capture, in the chat view and after expanding the tool output; the interactive TUI shows no more than the capture does, so a capture is the evidence.
  Rationale: the operator cannot be present, so the capture stands in for watching.
- A case that cannot run, such as a usage limit, is reported as not run with the error, never inferred from the docs.

## Constraints

- `CLAUDE.md`: agents run in interactive tmux, never `claude -p`; no production deploy; no push; markdown with semantic line breaks.
- PATTERN: no Go code changes, so no entry is touched; temp files go under `.scratch/`.
- Each step waits about 20 seconds before the next.

## Auto-mode assumptions

- A case that needs `--dangerously-skip-permissions` or Codex's bypass flags may use them, since the sessions are throwaway and run in this worktree.
- The Codex trust entry written to the operator's `~/.codex/config.toml` is unavoidable and is named in the report.

## Open risks

- Model usage limits may stop the Codex cases, as on 2026-10-25.
- A hook field the docs name may behave differently in the installed Claude Code version; the report records the observed behavior.
- One run per case gives no variance; a model's next step is one sample.

## Acceptance criteria

- The report holds all five cases, each with config, script, model reaction, pane capture, operator visibility and, for case 3, the idle-wake and timeout results.
- The report ends with a channel × engine table: delivers to the model, visible to the operator, works when idle.
- No file outside `docs/research/creel-hook-channels-<date>/` and `_lym/` is changed.
