# creel stage 2: orch and batten notices through creel

`creel-notices` · Note · feature, creel, orch, batten

## Goal

Creel stage 2: system notices go through creel.
The design is in `creel-core`'s body until that task lands, then in the creel package's `doc.go`.

## Operator decision (2026-10-15)

Creel is a prerequisite for Codex and is built in its three stages (operator, 2026-10-06).

## Scope (stage 2 of the creel design)

The orch watcher's and batten's notices and the watcher's injects go through creel, and the watcher's notice queue and idle probe go.
The watcher keeps only its cycle logic (tracking tokens, choosing `refresh`/`distill`/automatic cycles, waiting for the note, sequencing the reload, its state and lifecycle), and advances a cycle phase only on the orch's reply to the note request, or once creel confirms a command inject was typed.
The note request is mail, answered by post (`written <path>` or `defer`).

## Operator decision (2026-10-15): a note until creel has proven itself

This stage replaces paths that work today, so it stays a note, which never starts, until `creel-core` has landed and creel has proven itself; the operator then promotes it.
When promoted, it depends on `creel-core`.

## Operator decision (2026-10-17): nothing durable lives only in creel

This task extends `PATTERN-durable-vs-ephemeral-state` in the same commit as its code, with this text:
"Nothing a run, the orch or a landing depends on lives only under `.lyx`: decisions, landings, stop reports and board state are always in git (board, records, decision record), and never-tracked state such as the creel journal and the notice log is a delivery path or cache that a fresh machine starts without."
Background (orch, 2026-10-17): the creel journal sits in `<hub>/_board/.lyx/creel/journal/`, never tracked, so a move to another machine leaves undelivered mail behind; the strands it was meant for do not move either, and the orch reads landings from the board.

## Orch recommendation, not a decision (2026-10-17): the notice log goes

Once batten's notices are creel mail to the orch, a notice stays unread until the orch reads it, so batten's copy in the prime's `.lyx/notices.log` (`shedrun.NoticeLog`) is redundant.
The task removes it, and the orch role's fallback reads unread creel mail instead.
