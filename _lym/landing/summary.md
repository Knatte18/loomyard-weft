# Creel delivers mail to a working Claude strand via a PostToolUse hook

A Claude strand in a long turn now receives its creel mail at its next tool boundary, without ending the turn or losing a tool call in flight.
Before, a darn writer's mail waited for the turn to end, and `lym creel interrupt` would end the turn and lose a call in flight such as a gate run.
Interrupt stays for stopping an agent.

Closes #871

## What changes

- `creelengine` keeps one empty flag file per strand holding unread mail, in a new flag directory under the board-level `.lym/creel`.
  The store re-derives the flag set at the end of `Store.do`, so every journaled change moves the flags with the pool, and the daemon reconciles the directory at start.
  Programs get no flag, and the flags are derived state that is never journaled.
- The flag name is the strand name with every `:` turned into `.`, formed in `agentname`; the directory's name is a `lymdirs` constant and its path a `hubgeom` field.
- `claudeengine` renders a `PostToolUse` hook, with no matcher, into every hub Claude strand's settings.
  It is plain `sh`: it forms the flag name from `$LYM_STRAND_NAME` by parameter expansion alone and exits 0 at once when the flag is absent, so a call with no mail starts no process.
  When the flag stands it changes to `$CLAUDE_PROJECT_DIR` and runs `lym creel hook-read claude`.
  A standalone run, told no board, gets no hook.
- The new hidden verb `lym creel hook-read <engine>` claims the caller strand's mail through a new daemon verb and prints it as the engine's tool-boundary context: one `[creel] #<id> from <sender>: <body>` line per message, the line the idle path types for a notice.
  The mail is journaled read only after the output was written.
- The optional capability `shuttleengine.ToolBoundaryContextRenderer`, handed out by `shuttleengines`, keeps Claude's `hookSpecificOutput.additionalContext` JSON under `claudeengine`.
- Codex strands get no hook and keep the pane path.

## Design decisions to check

- **One claim for both paths.**
  The idle pane path now reserves a notice in the store before typing it and releases it after a busy send, so a hook claim and the pane path never both deliver one message.
  `markNoticeRead` drops the pane reservation, and a notice the hook claimed first is not typed.
- **What a claim hands out.**
  `Store.ClaimAtBoundary` takes unread, unreserved mail oldest first, agent mail included.
  A hold limits it to the hold owner's mail, a pause to none.
  It stops at an 8 KiB budget that counts the joining newlines, but always hands out the first message whole; the rest stays unread and the flag stands for the next boundary.
- **Weaker agent-mail guard.**
  The idle path sends agent mail as a reminder naming `lym creel read`, so an agent's text cannot pose as an instruction.
  The hook skips that and delivers the body; the line still opens with creel's stamped sender, and only mail the strand already holds is handed over.
- **Failure output.**
  `hook-read` prints nothing to standard output on a refusal or daemon error, writes the error to standard error and exits 1, since the provider parses a `PostToolUse` hook's standard output as JSON.
  Its daemon call times out after 10 seconds.
- **Known cost.**
  The flag stands while a hold mutes non-owner mail, and after a crash until the next daemon start reconciles it; in both cases the hook starts `lym` at each tool call.

## Docs

The `doc.go` of each changed package and `docs/overview.md` describe the flags, the claim and the hook.

## To check by hand

Tests stub Claude Code, so these were not run:

- Run a real Claude strand through a multi-tool turn, post to it from another strand, and confirm the `[creel]` line reaches the model at the next tool call and the message shows read afterwards.
- Confirm Claude Code passes `LYM_STRAND_NAME`, `CLAUDE_PROJECT_DIR` and the pane's `PATH` to the hook process, and passes 8 KiB of additional context unshortened.
