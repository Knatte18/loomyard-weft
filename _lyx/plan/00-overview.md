---
format: 5
approved: false
language: go
---

# Plan: fork-wait-noise — gates run in the foreground, so a fork wakes Merriam once

A Webster batch fork wakes Merriam once per backgrounded gate, because each backgrounded call ends the fork's turn and sends an interim notice.
This plan raises Claude Code's Bash timeout ceiling on every lyx-spawned session's launch and resume line, makes the raw-go steer the one statement of a foreground gate rule, and points every stencil, spec and refusal that named the background route at that rule instead.
Because foreground forks no longer send interim notices, the claude activity reading counts a session's fork transcripts, so loom's step idle kill and batten's quiet notice do not read a Merriam waiting on a working fork as quiet.
Merriam's template learns to read the remaining fallback, an interim notice from a call backgrounded at the ceiling, as no completion.

## Card Index

1 — bash-timeout-ceiling — launch and resume lines carry `BASH_MAX_TIMEOUT_MS` from one constant, and the raw-go steer becomes the foreground gate rule
2 — fork-activity — the turn-end activity reading counts fork transcripts, a pending fork tool call reading active up to the ceiling
3 — webster-fork-bodies — the implementer and verify-fix bodies run every gate in the foreground from their working directory and never background a call
4 — master-interim-notices — the Master template reads an interim fork or fixer notice as no completion and corrects its idle-turn-end sentence
5 — plan-route-foreground — the `verify-module-wide` refusal, its spec row, the plan stencil and `CLAUDE.md` name the foreground route
6 — orch-gate-rule-brief — the orch role stencil copies the gate rule into a test-running non-fork agent's brief

## Shared Decisions

- The gate rule is the raw-go steer in `internal/shuttleengine/claudeengine`; every stencil, spec and refusal points at "the gate rule in your system prompt" and never restates the timeout value, the quick-check rule or the `cd` rule.
- `BASH_MAX_TIMEOUT_MS` and the Bash tool's `timeout` are named in Go only under `internal/shuttleengine/claudeengine` (PATTERN-shuttle-provider-seam).
- No new PATTERN entry: the claims tests and the claudeengine tests pin the rule (decision record D8).
- No new top-level test: each card extends the test that already covers its surface (PATTERN-test-economy).

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
