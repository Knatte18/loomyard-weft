# Re-measure the batching hook once its transcript wait has landed

`batching-hook-remeasure-wait` · Task · research, claude, shuttle · running · Darn-Write

## Seen

The report `docs/research/turns/batching-hook-remeasure-2026-11-05` found the hook fired on 2 of 156 edit follows, while a replay of a recorded one fired. Task `batching-hook-live-gap` traced the gap: Claude Code writes a message to the transcript only after its calls start, so the hook of a fast Read, Grep, Glob or Edit call read a transcript that did not hold its own message yet. The hook now waits for the call, up to two seconds.

## Idea

Run the scripts of that report again over the runs landed after the fix: firings by breach and by tool, firings against breach turns, and the next message batched. Decide from those figures whether to keep the hook or remove it as the report recommended. Also price what the wait adds per call above `context_tokens`.