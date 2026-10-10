# Burler runs its round on MultiLLMProducer

`burler-multi-llm` · Note · enhancement, burler, shed, stencil

## Goal

Burler's review round runs on `MultiLLMProducer` instead of its own two-agent wiring (operator, 2026-10-08).

## Today

Burler's round has run two agents ad hoc since 572ab84e8: `roundHalves` in `internal/burlerengine/engine.go` starts the reviewer and the fixer, and the fixer waits on the review through `lyx burler await-review`.
The round wires six stencils (two orchestrator templates, three step files, a focus directive) plus per-segment rubrics through Go code spread over several packages.

## Wanted

- The fixer is the `chair` and the reviewer `advisor-1`, declared in one seat table.
- The ad hoc wiring (`roundHalves`, `await-review`) goes; the round's outcome, review files and Bouncer contract stay as they are.
- Fanned rounds (`discussion_fan`, `plan_fan`) keep working; whether a fan's lenses become advisor seats or stay forks of the reviewer is decided in Discussion.
