# Clean up PATTERN.md: far too many entries

`pattern-cleanup` · Note · enhancement, stencil

## Seen (2026-10-24)

PATTERN.md was already over its headroom line (15651 bytes against 15564, 95% of 16384) before `gate-runs-once` added 510 bytes of entries; that run got through only by shortening 20 other entries, recorded as `changes` decisions.
The overview is inlined in every spawned agent's prompt, so its size is paid on every run.

## Idea (operator, 2026-10-24)

PATTERN has far too many entries and must be cleaned up: a dedicated agent first reviews which entries can be merged, moved to their background files or package `doc.go`, or dropped, and proposes a smaller set to the operator before any task.