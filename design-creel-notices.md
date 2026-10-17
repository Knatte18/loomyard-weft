# creel stage 2: orch and batten notices through creel

`creel-notices` · Task · feature, creel, orch, batten, shed

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

## Promoted (operator, 2026-10-17)

Creel proved itself: the C13 re-test (`docs/research/creel-audit-2026-10-17/`), the creel CLI fixes, and the llm smoke `TestSmokeCreelClaudeStrand` green on the operator's run.
The operator promoted this stage on 2026-10-17, with the reminder change below folded in.

## Reminders carry no content (folded in from note `creel-reminder-no-subject`)

### Seen (orch, 2026-10-17)

A demo post from `ly:orch` to a fresh Claude strand `ly:demo` was reminded as:
`[creel] 1 unread: #3 from ly:orch «Hello from ly:orch via creel. Please reply to this message with: lyx creel post » — run lyx creel read`.
This is the documented design (`internal/creelengine/doc.go`, the reminder paragraph): the subject is the body's first line, cut at 80 runes, inside « ».
A message without a separate first line therefore shows its content in the reminder, and the receiving agent can act on it without running `lyx creel read`.

### Operator decision (2026-10-17)

Creel's reminder carries no message content, only the count, ids and senders; urgent content goes by inject.

### Operator view, not a decision (2026-10-17)

The receiver has to ask for the content actively with `lyx creel read`.
A reminder can reach an agent in the middle of a run, busy with something else: it should only learn that one or more messages wait, and choose itself when it has the time and capacity to read them.
Part of a message arriving with the reminder is clutter and confuses the agent.
Something the agent must read at once goes by `lyx creel inject` instead (with `--interrupt` when the running turn must stop first).

### Idea

- The reminder names only the count, ids and senders, such as `[creel] 1 unread: #3 from ly:orch — run lyx creel read`.
- `lyx creel log` keeps its subjects, since only the operator and the orch read the journal.

### Open

- The reminder is typed only at idle, but in Claude a typed line starts a new turn, so even a content-free reminder takes the agent's next turn: whether stencils should tell agents that a reminder may be left until their current task is done.
- Whether the journal's `subject` field stays as the body's first line or goes too.
