# Usage and cost read through the engine

`engine-usage-readers` · Task · enhancement, shuttle, claude · running · Discussion-Burler-Fix

## Goal

A task of the Codex-readiness series (see `engine-registry` for the operator decision): token use and cost read through each run's engine, not straight from Claude's transcripts.

## Seen (2026-10-15)

- `tools/tokencount/main.go` reads `~/.claude/projects`; `cost.go` holds a Claude price table; `count.go` carries its own copy of the Claude transcript usage parser, duplicating `claudeengine/usage.go`; `calibrate.go` reads `CLAUDE.md`.

## Scope

- A neutral per-message usage record and a transcript locator capability in `shuttleengine`; `claudeengine` implements them.
- `tokencount` reads through each run's engine and deletes its own parser.
- Prices move into each engine behind a capability.
- Calibration reads the engine's instruction files.

## Out

Note `report-timing` itself; it should wait on this task if it is to report token totals.
