---
format: 5
approved: true
language: go
---

# Plan: shuttle-hook-first

Shuttle reads a Claude session's state from its hook lines whenever they settle it, and reads the pane only for what no hook carries: startup gates, the input box and its drafts, a menu the operator opened, and operator activity.
The first card records every `SessionStart` source and `PermissionRequest` and captures a real hook corpus, which the parser and fold cards test against.
Folded in: a bounded clear of a lone stray character in an idle input box (#554), batten's busy notice quoting that box, and a tail-bounded read of fork transcripts.
Card 7 is built only when card 1's corpus shows `SessionStart(startup)` landing on its own; otherwise it is reported as a blocker and skipped.

## Card Index

1 — hook-recording-capture — record SessionStart for every source and PermissionRequest on every run, and capture the real hook corpus
2 — session-start-source — add the provider-neutral session-start source type and map Claude's source strings to it
3 — idle-fresh-fold — add the idle-fresh fold state a session start sets, read as ready and as idle
4 — permission-request-ask — read PermissionRequest as an ask, map every Notification type, release an ask on an interrupt
5 — fork-transcript-tail — read a fork transcript's pending call from its newest tool-calling message only
6 — clear-compact-confirm — confirm /clear and /compact by their matching SessionStart line past the pre-send offset
7 — startup-needs-session-start — start a run only once a SessionStart line and the ready pane both hold
8 — dead-session-ends-run — end a run died or done when the registry or a SessionEnd proves the process gone
9 — stray-input-reader — add the StrayInputReader capability and Claude's lone stray-character judgment
10 — stray-character-clear — clear a lone stray character once behind a full guard reading, then read it as residue
11 — batten-busy-notice-box — carry the refused resume's box text in batten's driver-busy notice

## Shared Decisions

- Claude's hook names, payload fields, source strings, notification types and the stray-character judgment live only in `claudeengine`; `shuttleengine` sees `SessionStartSource`, signal kinds and box readings (`PATTERN-shuttle-provider-seam`).
- An engine without `SessionSignalParser`, `SessionProber`, `StrayInputReader` or a launch that records session starts behaves exactly as today on every path the plan touches.
- Tests extend the existing tables; a card that adds a top-level test lists `cmd/lyx/testdata/test-budget.yaml`.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
