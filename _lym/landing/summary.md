# Report which hook channels deliver a creel post and show to the operator

Adds `docs/research/creel-hook-channels-2026-10-28/report.md`, the result of five hook-channel cases run live on throwaway interactive sessions with hand-written hook scripts standing in for creel.
No lym code, plugin or setting changed.

## What the report shows

- Claude `PostToolUse` delivers mid-turn, and its `systemMessage` shows in the chat as `PostToolUse:Bash says: …`;
  Haiku received the message but declined to act on it, though `CLAUDE.md` told it to.
- Claude `Stop` with `decision: "block"` delivers, is shown as `Stop hook error: <reason>`, and was acted on both times.
- Claude `asyncRewake` at `Stop` woke an idle session 0.3 s after the file appeared and started a turn by itself, left a half-typed draft in the input box untouched, and showed only a bare `Stop hook feedback` line;
  the hook's timeout killed it at the configured 45 s with no wake and nothing on screen.
- Claude `MessageDisplay` shows the line in the chat only; the model reported nothing in its context and the expanded transcript omits it.
- Codex `post_tool_use` delivers, shows as `↳ Hook · …`, and the model said it would act; the hook input has the Claude shape.

The report ends with the channel × engine table the task asked for, with pane captures and the hook scripts per case.

## Design decisions

- Case 3 was run twice, once as first written and once with an unsubmitted draft in the input box, per the parent's recorded decision.
- Turn ends from case 2 on were read from hook log files, per the parent's recorded decision; case 1 had waited on the pane and was captured by hand.
- `pre_model_request` was not run because `post_tool_use` delivered.
  Codex idle delivery was not asked for and was not run.
- Cases 1 and 5 set `additionalContext` and `systemMessage` together, so they do not say which field carried the model-side delivery.
- The Codex run wrote one trust entry to the operator's `~/.codex/config.toml`.

## Testing

Documentation only; nothing builds or runs from the changed path.
