# Antigravity CLI engine spike

`antigravity-engine-spike` · Note · research, shuttle

## Seen (orch, 2026-11-17)

The desk study `docs/research/gemini-antigravity-engine-study-2026-11-17.md` (`246c9146`) judged Antigravity CLI (`agy`) the better Google engine target.
Its terms aim at third-party services reselling access, not at the operator driving the CLI with a tool they wrote, as with Codex and Claude (operator, 2026-11-17).
Several of its readings are undocumented and can only be learnt hands-on.

## Idea

A hands-on spike like Codex's of 2026-10-24, run as darn docs, that installs `agy` and runs it in a tmux pane on Neve.
It answers the study's open points:

- the busy, idle, trust and interrupt screens, and bracketed paste plus Enter through `paste-buffer -p`;
- whether the keyring login works in a lym pane;
- the hook payloads for SessionStart, PreToolUse, PostToolUse, PostInvocation and Stop (`continue`, `fullyIdle`);
- the transcript's on-disk layout and token fields;
- whether a conversation id can be chosen up front, and whether the `self` subagent inherits the caller's history.

Cap: one machine, one model, one short session per point, no lym code changed.
