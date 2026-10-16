# Creel gets its own sandbox suite

`creel-sandbox-suite` · Task · enhancement, creel, shuttle · running · Plan-Burler-Review

## Seen

The creel sandbox scenarios live in the shuttle suite: `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` holds S9 (creel delivers mail and injects into a hand-added strand) and C1 to C13, each with `**Covers:** creel`.
Task `creel-stage1-fixes` put C1 to C13 there because S9 already was, by its own Discussion's choice (D9, "beside S9"), not by an operator decision.
Every other module with sandbox scenarios has its own suite file (`SANDBOX-REED-SUITE.md`, `SANDBOX-FABRIC-SUITE.md`, `SANDBOX-BURLER-SUITE.md` and the others under `tools/sandbox/`).

## Operator decision (2026-10-16)

Creel gets its own sandbox suite.

## Idea

Agent's suggestion: a new `tools/sandbox/SANDBOX-CREEL-SUITE.md` with the black-box rule, the controlled tmux and creel exceptions, pre-conditions and session-log format creel needs, holding S9 and C1 to C13 moved out of the shuttle suite with their ids kept; the shuttle suite keeps only shuttle scenarios; `tools/sandbox/suite.go` and its tests and the `PATTERN-sandbox-coverage` guard follow the move.
