# Creel reminders carry no message content

`creel-reminder-no-subject` · Note · enhancement, creel

## Seen (orch, 2026-10-17)

A demo post from `ly:orch` to a fresh Claude strand `ly:demo` was reminded as:
`[creel] 1 unread: #3 from ly:orch «Hello from ly:orch via creel. Please reply to this message with: lyx creel post » — run lyx creel read`.
This is the documented design (`internal/creelengine/doc.go`, the reminder paragraph): the subject is the body's first line, cut at 80 runes, inside « ».
A message without a separate first line therefore shows its content in the reminder, and the receiving agent can act on it without running `lyx creel read`.

## Operator decision (2026-10-17)

Creel's reminder carries no message content, only the count, ids and senders; urgent content goes by inject.

## Operator view, not a decision (2026-10-17)

The reminder must not carry the message's content.
The receiver has to ask for it actively with `lyx creel read`.
A reminder can reach an agent in the middle of a run, busy with something else: it should only learn that one or more messages wait, and choose itself when it has the time and capacity to read them.
Part of a message arriving with the reminder is clutter and confuses the agent.
Something the agent must read at once goes by `lyx creel inject` instead (with `--interrupt` when the running turn must stop first).

## Idea

- The reminder names only the count, ids and senders, such as `[creel] 1 unread: #3 from ly:orch — run lyx creel read`.
- `lyx creel log` keeps its subjects, since only the operator and the orch read the journal.

## Open

- The reminder is typed only at idle, but in Claude a typed line starts a new turn, so even a content-free reminder takes the agent's next turn: whether stencils should tell agents that a reminder may be left until their current task is done.

- Whether the journal's `subject` field stays as the body's first line or goes too.
