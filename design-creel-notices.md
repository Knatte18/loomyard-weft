# creel stage 2: orch and batten notices through creel

`creel-notices` · Task · feature, creel, orch, batten

Depends on: [`creel-core`](design-creel-core.md)

## Goal

Creel stage 2: system notices go through creel.
The design is in `creel-core`'s body until that task lands, then in the creel package's `doc.go`.

## Operator decision (2026-10-15)

Creel is a prerequisite for Codex and is built in its three stages (operator, 2026-10-06).

## Scope (stage 2 of the creel design)

The orch watcher's and batten's notices and the watcher's injects go through creel, and the watcher's notice queue and idle probe go.
The watcher keeps only its cycle logic (tracking tokens, choosing `refresh`/`distill`/automatic cycles, waiting for the note, sequencing the reload, its state and lifecycle), and advances a cycle phase only on the orch's reply to the note request, or once creel confirms a command inject was typed.
The note request is mail, answered by post (`written <path>` or `defer`).

## Operator decision (2026-10-15): no auto-start

This stage replaces paths that work today, so it starts only on the operator's word once `creel-core` has landed and creel has proven itself, never on a newly-ready auto-start.
